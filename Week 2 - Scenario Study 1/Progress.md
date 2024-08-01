### Progress

<hr>

`progress`는 팔로워들의 복제 상태들을 추적하는데 사용한다.

- `matched`: 리더와 일치하는 로그 인덱스
- `next_idx`: 다음 로그가 올 인덱스
- `state`: 복제상태. 하트비트 받기용, 스냅샷 처리용, 로그 복제용 이렇게 나뉜다. 
- `paused`
- `pending_snapshot`: 보류중인 스냅샷 인덱스
- `pending_request_snapshot`: 요청한 스냅샷 인덱스
- `recent_active`: 최근 메세지를 수신했는가
- `ins`
- `commit_group_id` : 그룹 커밋에 사용
- `committed_index`: raft_log에서 커밋된 인덱스

이러한 필드를 사용하여 복제 상태를 추적한다.