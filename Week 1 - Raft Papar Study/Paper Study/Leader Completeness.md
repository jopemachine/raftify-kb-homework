만약 log entry가 주어진 term에서 commit 된다면, 해당 entry는 더 높은 term에서도 leader log에 존재한다


## 증명 (귀류법)
[[Leader Append-Only]]와 [[Log Matching]] Property 가 성립한다면 Leader Completeness 가 성립한다
### 가정
- 리더가 term T에서 log entry L을 커밋했다.
- 미래의 term U 에서 리더에 log entry L이 저장되지 않았다

### 증명
1. 리더는 log entry 를 지우거나 덮어쓰지 않는다. 따라서 log entry L은 term U에서 리더의 로그에 누락되어 있다
2. Term T에서 Leader(이때 Leader를 Leader T라고 하자) 는 과반수 node에 log entry L을 복제했다.
3. Term U에서 Leader(이때 Leader를 Leader U라고 하자) 는 과반수 node로부터 투표를 받았다
4. 따라서 적어도 한 node는 Term T에 log entry L을 복사하고, Term U에 Leader U에 투표했다. (비둘기 집의 원리).이 node를 voter라고 하자
5. Voter는 Leader U에 투표할 때까지 log entry L을 저장하고 있었다. 왜냐하면 log entry가 제거되는 경우는 Follower와 Leader가 충돌되는 경우 뿐인데, Leader는 log entry를 수정하지 않으므로 log entry L은 Leader와 Follower모두 갖고 있어 충돌되지 않는다.
6. Voter는 Leader U에 투표했으므로 Leader U의 로그는 적어도 voter의 log보다 up-to-date이다.
	1. 만약 voter와 Leader U가 마지막 index와 term이 같다면 Log Matching Property에 의해 Leader U의 로그와 투표자의 log는 동일하다. 따라서 Leader U는 log entry L을 저장해야 한다. 모순
	2. 만약 voter보다 Leader U의 index가 크다고 하자. voter는 term T에서 commit된 값을 갖고 있으므로 voter가 가질 수 있는 term의 최소값은 T이다. Log Matching Property에 의해 Leader U는 이전에 commit된 값을 포함해야 한다. 즉, log entry L을 저장해야 한다. 모순
7. 따라서 T보다 큰 모든 term의 리더는 T에서 commit된 모든 log entry를 가져야 한다.
