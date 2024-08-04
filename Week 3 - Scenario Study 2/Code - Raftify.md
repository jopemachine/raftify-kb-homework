- tikv/raft-rs 소스코드를 최대한 그대로 활용하되, 필요한 부분은 커스텀해서 사용 중


### Example/memstore/dynamic-members/src/main.rs

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
    - HashStore: read/write 인터페이스를 제공하는 일종의 hash map db.