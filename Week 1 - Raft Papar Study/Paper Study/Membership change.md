장애로 인한 node 교체, Cluster 내 node 수 변경 등 여러 요인으로 Configuration[^1]이 변경해야할 수 있다
Configuration 변경이 offline에서 진행된다면 configuration이 진행되는 동안 서비스를 제공할 수 없다
Raft는 고가용성을 위해 Configuration 변경을 online에서 진행한다

## Procedure
모든 node의 configuration을 한번에 안전하게 업데이트하는 방법은 존재하지 않는다
- 각 node가 configuration이 업데이트되는 시간이 완전히 일치할 수 없기 때문에 같은 Term에 동시에 두 Leader가 존재할 수 있다. 이는 하나의 Term에 하나의 Leader만 존재한다는 Raft의 가정에 위반된다

따라서 2단계로 나누어 Configuration을 업데이트 한다.
![[Pasted image 20240725200003.png]]
Stage 1. [[Joint Consensus]]
- 안전성에 대한 보장없이 각 노드들이 서비스를 제공하면서 Configuration을 업데이트할 수 있음
Stage 2. Update to the new configuration

## Procedure
1. 리더에게 new configuration으로 변경하도록 요청한다
2. 리더는 joint consensus을 위한 새로운 configuration과 해당하는 log entry를 생성한다
3. follower들에게 joint consensus에 해당하는 log entry를 복사한다
	1. joint consensus에 해당하는 log entry가 commit되면, old configuration, new configuration에 해당하는 모든 node의 동의를 얻어 의사 결정을 내린다
4. New configuration에 해당하는 log entry를 만들어 복사한다
	1. New configuration은 node에서 발견되는 대로 적용한다
5. New configuration에 해당하는 log entry가 commit되면 old configuration에만 존재하는 node는 종료된다

### Issue 1. 초기 새로 추가되는 node의 log entry가 비어 있다
새로 추가되는 node는 log entry를 갖고 있지 않기 때문에 log entry를 복사해야 한다.
이로 발생하는 딜레이를 최소화하기 위해 새로 추가되는 node는 바로 투표가 가능하지 않고, leader의 log entry만 복사한다.
이후 configuration 업데이트가 진행된다

### Issue 2. Leader가 new configuration에 포함되지 않을 수 있다.
이 경우 Leader는 new configuration을 commit 한 후 Follower로 변경된다.
Follower로 변경되기전까지 Leader는 자신을 포함하지 않은 시스템을 관리하게 된다.

### Issue 3. New configuration에 포함되지 않은 node들이 가용성을 떨어뜨린다
만약 Leader가 old configuration과 new configuration에 모두 속해있다고 하자.
New configuartion에 포함되지 않은 node는 heartbeat를 받지 않아 Leader Election을 시작한다.
그러면 새로운 term으로 RequestVote RPC를 보내고, 이는 현재 Leader를 Follower 로 만든다. 그리고 이 과정이 무한히 반복된다

이 문제를 방지하기 위해 Follower는 현재 Leader가 존재한다고 판단할 시 RequestVote RPC를 무시한다

[^1]: 합의 알고리즘 에 참여하고 있는 node 집합