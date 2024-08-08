- tikv/raft-rs 소스코드를 최대한 그대로 활용하되, 필요한 부분은 커스텀해서 사용 중
- raft_service 에 대한 코드는 tonic 이 .proto 파일을 보고 자동으로 생성해준다. (raft_service.rs)
- 메타데이터는 lmdb 를 러스트로 랩핑해주는 라이브러리 써서 안전하게 관리.

### 모듈별 실행 방법

```
make build
./target/debug/{module name}

// 예)
./target/debug/memstore-dynamic-members
./target/debug/raftify-client-example
./target/debug/raftify-cli
...
```

- 빌드 관련 내용은 모두 Cargo.toml 에 들어가있음!
- 실행 시 cluster_config.toml 파일에 적은 내용을 로드하는 부분은 `load_peers()` 함수에 있음.

### 예제 코드 구경 - Example/memstore/dynamic-members/src/main.rs

```rust
#[macro_use] // 아래 crate의 매크로 import.
extern crate slog;
extern crate slog_async;
extern crate slog_term;

use actix_web::{web, App, HttpServer}; // 웹서버를 위해
use raftify::{ // raftify 에서 필요한 것만 import.
    raft::{formatter::set_custom_formatter, logger::Slogger},
    CustomFormatter, Raft as Raft_,
};
use slog::Drain; // slog의 매크로만 불러왔기 때문에 Drain 은 따로 불러와야 함
use slog_envlogger::LogBuilder;
use std::sync::Arc;
use structopt::StructOpt; // Command line argument 를 받아서 처리하기 위해.

use example_harness::config::build_config;
use memstore_example_harness::{
    state_machine::{HashStore, LogEntry},
    web_server_api::{
        campaign, debug, demote, get, leader_id, leave, leave_joint, peers, put, snapshot,
        transfer_leader,
    },
};

type Raft = Raft_<LogEntry, HashStore>;

#[derive(Debug, StructOpt)]
struct Options { // CLI 로 example 실행할 때 파라미터로 넣어줬던 애들!
    #[structopt(long)]
    raft_addr: String,
    #[structopt(long)]
    peer_addr: Option<String>,
    #[structopt(long)]
    web_server: Option<String>,
}

#[actix_rt::main] // 비동기 actor model 런타임 프레임워크
async fn main() -> std::result::Result<(), Box<dyn std::error::Error>> {
    color_backtrace::install();

    let decorator = slog_term::TermDecorator::new().build();
    let drain = slog_term::FullFormat::new(decorator).build().fuse();
    // 빠르고 효율적인 로깅을 위해 slog 사용. 예) 직렬화 및 I/O 작업과 같이 비용이 많이 드는 작업은 다른 스레드한테 토스.
    let drain = slog_async::Async::new(drain).build().fuse();

    let mut builder = LogBuilder::new(drain);
    builder = builder.filter(None, slog::FilterLevel::Debug);

    if let Ok(s) = std::env::var("RUST_LOG") {
        builder = builder.parse(&s);
    }
    let drain = builder.build().fuse();

    let logger = Arc::new(Slogger {
        slog: slog::Logger::root(drain, o!()),
    });

    set_custom_formatter(CustomFormatter::<LogEntry, HashStore>::new());
    
    let options = Options::from_args(); // CLI 파라미터로 넘긴 값 받아오고
    let store = HashStore::new(); // 노드의 상태 머신을 생성. (로그를 저장하는 그 storage 와 다르다.)

    let mut cfg = build_config(); // raft configuration 생성

    let (raft, raft_handle) = match options.peer_addr {
        Some(peer_addr) => { // 피어 주소가 주어졌다면 팔로워 모드로 노드 실행.
            log::info!("Running in Follower mode");

            let ticket = Raft::request_id(options.raft_addr.clone(), peer_addr.clone())
                .await
                .unwrap();
            let node_id = ticket.reserved_id;
            cfg.initial_peers = Some(ticket.peers.clone().into());

            let raft = Raft::bootstrap(
                node_id,
                options.raft_addr,
                store.clone(),
                cfg.clone(),
                logger.clone(),
            )?;

            let handle = tokio::spawn(raft.clone().run());
            raft.add_peers(ticket.peers.clone()).await;
            raft.join_cluster(vec![ticket]).await;

            (raft, handle)
        }
        None => { // 그렇지 않으면 리더 모드로 노드 실행.
            log::info!("Bootstrap a Raft Cluster");
            let raft = Raft::bootstrap(1, options.raft_addr, store.clone(), cfg, logger.clone())?;
            let handle = tokio::spawn(raft.clone().run());
            (raft, handle)
        }
    };

    if let Some(addr) = options.web_server { // 웹서버 실행
        let _web_server = tokio::spawn(
            HttpServer::new(move || {
                App::new()
                    .app_data(web::Data::new((store.clone(), raft.clone())))
                    .service(put)
                    .service(get)
                    .service(leave)
                    .service(debug)
                    .service(peers)
                    .service(snapshot)
                    .service(leader_id)
                    .service(leave_joint)
                    .service(transfer_leader)
                    .service(campaign)
                    .service(demote)
            })
            .bind(addr)
            .unwrap()
            .run(),
        );
    }

    let result = tokio::try_join!(raft_handle)?;
    result.0?;
    Ok(())
}

```

```rust
#[derive(Clone)]
pub struct Raft<LogEntry: AbstractLogEntry + 'static, FSM: AbstractStateMachine + Clone + 'static> {
    pub raft_node: RaftNode<LogEntry, FSM>,
    pub raft_server: RaftServer<LogEntry, FSM>,
    pub tx_server: mpsc::Sender<ServerRequestMsg<LogEntry, FSM>>,
    pub logger: Arc<dyn Logger>,
}
```

- Raftify 의 클라이언트는 AbstractLogEntry 를 구현하는 LogEntry 타입과 AbstractStateMachine 을 구현하는 FSM 타입 인스턴스를 만들어서 넘겨야 함.
  - 예제에서는 LogEntry, HashStore 를 넘겨주고 있음.
    - LogEntry: key-value 쌍. ""이 key 를 가진 데이터를 이 value로 업데이트""
    - HashStore: read/write 인터페이스를 제공하는 일종의 hash map db. 이것이 state machine 의 state 가 된다.

#### raft_bootstrapper 에서 하는 일

- RaftNode, RaftServer 를 하나씩 띄운다. 
  - RaftNode: 논문의 그 raft node. 합의 알고리즘에 관여함.
    - RaftNode.run() 안에서는 무한 루프를 돈다. 이게 저번에 봤던 RaftLoop!!
  - RaftServer: 클라이언트와 통신하는 역할.
- `run()` 함수에서 셋 중 하나가 일어나면 클러스터가 종료된다.
  - Ctrl + C 시그널 받거나
  - RaftNode 가 종료되거나
  - RaftServer 가 종료되거나.

# RaftNode, RaftNodeCore

- RawNode 는 thread-unsafe 하다.
  - 이걸 thread-safe 하게 만들기 위해 랩핑한 것이 RaftNodeCore

#### LocalRequestMsg

- 클라이언트 -> 서버로 요청했을 때. 메소드 안에서 일어나는 일에 대한 메세지.

- RaftNodeCore 에 대한 접근을 atomic하게 보장하기 위한 것이 RaftNode.

  - RaftNodeCore 에 대한 소유권을 비동기 태스크로 넘겨버리고 더이상 다른 애들이 접근 못하게 OneShotMutext 를 사용함.

  - RaftNode.(method_call) 은 RaftNodeCore.run 으로 넘어감.

    - RaftNode.tx_local.send(LocalRequestMsg::{메세지타입}) 형태로 호출해서 자기자신에게 메세지를 보낸다.

    - 메세지를 받으면 handle_local_request_msg 에서 메세지 타입별로 처리.

      ```rust
        // 이걸 호출하는 애들은 다 ReftNodeCore 에 대한 소유권을 갖고 있지 않은 애들.  
          pub async fn is_leader(&self) -> bool { // immutable 하게 자기 자신을 참조
              let (tx, rx) = oneshot::channel(); // oneshot 채널 하나 생성. oneshot 채널에는 하나의 태스크가 딱 1번 send 하고, 하나의 receiver 가 1번 consume 할 수 있다. 그게 각각 `tx` 와  `rx`.
              self.tx_local
                  // self.tx_local 은 mpsc::Sender<T> 타입이다. 이 타입은 이걸 클론해서 여러 owner 가 채널을 공유하기 위해 만들어졌기 때문에 _동시에_ 여러 스레드나 태스크에서 사용할 수 있다. (오~)
                  .send(LocalRequestMsg::IsLeader { tx_msg: tx }) 
                  // mpsc::Sender.send 를 호출하기 위해서는 mutable 참조가 필요없기 때문에, self 를 immutable하게 참조하고 있는 이 함수에서도 send 를 호출할 수 있다.
                  .await // 비동기 태스크 기다리고
                  .unwrap(); // Result 값을 꺼내고 Ok 인 경우에는 그냥 버린다. 정상 리턴값은 딱히 중요하지 않고 Error 를 감지하는게 목표일 때 이렇게 자주 쓴다고 함. 하긴... 여기서 중요한건 send() 함수의 리턴값 자체가 아니다. 뭘 던졌느냐가 중요함.
              let resp = rx.await.unwrap(); // 이제 Sender 가 던진 값이 뭔지 기다린 뒤 unwrap 함.
      
              match resp { // 받은 값이
                  LocalResponseMsg::IsLeader { is_leader } => is_leader, // is_leader 필드를 가진 LocalResponseMsg::IsLeader 라면, is_leader 값을 리턴.
                  _ => unreachable!(), // 안돼~~~
              }
          }
      ```
      
      - 아니 여기서 도대체 어디서 내가 리더인지 확인하는거지 ?  ?라고 생각했으나
      
      - 잘 보면 Sender 가 is_leader 안에서 만든 채널의 sender 가 아니라, **self.tx_local** 의 sender 다. 즉, 부트스트랩 할 때 만들어놨던 그 mpsc 채널로 보내는 것 !! ! ! !!
      
      - 그리고 self.tx_local.send 한 것은 handle_local_request_msg 에서 받아서 처리하는데... .그 안을 잘 보면 여기서 is_leader 안에서 만든 원샷 채널의 `tx.send` 를 호출한다 ! !
      
      - ```rust
        async fn handle_local_request_msg(
                &mut self, // 자기자신에 대한 mutable 참조
                message: LocalRequestMsg<LogEntry, FSM>, // 내가 받은 메세지~
            ) -> Result<()> {
                match message {
                    LocalRequestMsg::IsLeader { tx_msg } => {
                        tx_msg // self.tx_local.send 호출한 함수에서 만든 원샷 채널에 대한 sender
                            .send(LocalResponseMsg::IsLeader { // 에게 보낸다.
                                is_leader: self.is_leader(), // 내가 리더인지 담아서
                            })
                            .unwrap();
                    }
        ```

- 정리하면 이렇게 됨

![LocalReqeustMsg](../LocalRequestMsg 처리 과정.jpg)

#### ServerRequestMsg

- 서버끼리 gRPC 요청을 처리할 때. raft_server.rs 에서 일어나는 일에 대한 메세지
- LocalRequest 와 동일한 방식으로(mpsc::channel 에 쓰고, 거기서 oneshot::channel 에 쓰고.) 처리함. 단, 처리하는 함수는 handle_server_request_msg.