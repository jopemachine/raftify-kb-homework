--- 
###### 임기
- Raft는 시간을 `term`이라고 하는 임의의 길이로 쪼갬
- `term`은 연속된 정수들로서 논리적인 시계 역할
- 각각의 서버들에 `current term`을 저장해 사용함으로써 stale된 leader를 탐지
- `current term`은 서버들이 통신할 때 마다 교환
- 서버의 `current term`이 다른 서버의 `current term`보다 작다면 해당 `current term`의 값을 더 큰 값으로 업데이트
- candinate or leader의 `term` 값이 구식 -> follower로 복귀
- 구식 `term` 값을 포함한 요청 -> 해당 요청 거절
![[Pasted image 20240720051745.png]]

- 시간은 여러 term으로 쪼갬, term의 시작마다 선거
- 선거가 성공적으로 끝나면 leader가 그 term의 끝까지 클러스터를 관리
- 선거가 실패하는 경우 leader 없이 term이 종료