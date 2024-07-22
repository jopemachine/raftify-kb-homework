### 하트비트

<hr>

리더는 클러스터 내 모든 노드들에게 주기적으로 하트비트를 보내 자신이 리더임을 알림과 동시에 살아있음을 알린다.

팔로워들은 리더의 하트비트를 일정 시간 동안 받지 못하면 [Election timeout](./Election timeout.md)이 발생하여 자신이 [후보자](./Leader Election.md)가 된다.