[Raft](https://raft.github.io/)는 tikv, etcd, MongoDB 등 다양한 분산 시스템에서 사용되는 실용적인 합의 알고리즘이다.

기존의 [Paxos](https://ko.wikipedia.org/wiki/%ED%8C%A9%EC%86%8C%EC%8A%A4_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99)) 같은 합의 알고리즘에 비해 훨씬 이해하거나 구현하기 쉬우면서 이에 필적할만한 성능을 제공한다. 특히 _이해 가능성_(_Understandability_)은 Raft 알고리즘의 가장 중요한 목표이기도 하다.

Raft는 크게 두 가지 접근을 통해 이해 가능성을 달성한다.

1. **문제를 나눠서 접근하기**: `Leader Election`, `Log Replication`, `Membership changes`.
2. **고려해야 하는 상태의 수를 줄여 상태 공간 단순화 하기**.


### Raft만의 특징

Raft는 다른 분산 시스템 알고리즘들과 비슷하게 작동하지만 아래는 Raft만의 특징들이다.

- `Strong Leader`: Raft는 다른 합의 알고리즘들에 비해 리더에게 더 많은 권한을 부여한다. 예를 들어 로그 엔트리들은 리더에서 다른 팔로워 서버들로 단방향으로 흐르게 된다. 이러한 단방향 흐름은 복제된 로그들을 관리하기 수월하고 이해하기 쉽게 만들어준다.
    
- `Leader Election`: Raft는 리더를 선출할 때 무작위 타이머를 사용한다. 무작위 타이머는 아주 약간의 구현 추가로 충돌 문제를 단순하고 빠르게 해결해준다.
    
- `Membership Changes`: Raft의 메커니즘은 클러스터 내 서버들을 변경할 때 업데이트 기간 동안 서로 다른 설정들이 overlap 되는 `Joint consensus`라는 새로운 접근 방식을 사용한다. 이러한 접근 방식은 클러스터가 운영 중일 동안 설정 업데이트를 가능하도록 만들어준다.