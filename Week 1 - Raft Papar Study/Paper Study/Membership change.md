Cluster의 _configuration_ 이 고정되었다고 가정했지만, 실제로는 configuration을 바꿀 때가 간혹 있다.
Cluster 내에 모든 서버들을 종료하고 configuration 파일을 업데이하트 재시작하는 방법이 쉽지만 unavailable 상태가 되며, 수동으로 바꾸는 것은 위험한 운영 장애가 발생할 수 있다.

Configuration 파일 변경 자동화를 Raft 합의 알고리즘의 기능으로 사용할 수 있다.
업데이트를 진행하는 동안 두 개의 리더가 같은 term에서 선출되는 시점이 있으면 안된다.
Atomic하게 모든 서버들은 한 번에 변경하는 것은 불가능하기 때문에 업데이트 기간 동안 두 부분(Two Disjoint Marjorities)으로 나눠서 하면 된다.
안전성을 보장하기 위해 Two-phase 접근 방식을 사용해야 한다.

Raft에서는 _joint consensus_ 중간 단계로 우선 전환한 후, consensus가 커밋된 후 새로운 configuration으로 전환한다. Joint Consensus는 old configuration과 new configuration 둘 다를 합쳐 구성한다.
- Log entry들은 양쪽 설정의 모든 서버들에 복제된다.
- 어떠한 설정의 서버가 leader로 선출될 수 있다.
- Leader Election이나 entry 커밋을 위해 old 서버들과 new 서버들 각각은 다수결의 서버들의 합의가 요구된다

