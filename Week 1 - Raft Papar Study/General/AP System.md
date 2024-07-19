#### A : [[Availability.md|Availabilty]]
- 가용성
- 모든 요청이 성공적으로 처리
#### P : [[Partition Tolerance.md|Partition Tolerance]]
- 분할 내성
- 네트워크 분할이 발생해도 시스템이 정상적으로 작동
--- 
- C를 제외한 시스템
- 일관성을 희생시키면서 가용성과 분할 내성을 만족
- 분할 발생 -> 일부 노드 오래된 데이터
- 분할 해결 -> 노드를 동기화 -> 일관성 문제 만족
- CassandraDB는 AP 저장소