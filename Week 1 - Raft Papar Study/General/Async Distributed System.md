--- 
###### [[Sync vs Async|비동기식]] 분산 시스템
- 비동기 네트워크 환경에서는 실패 가능한 노드가 단 하나라도 존재한다 존재한다면, Safety와 Liveness 둘 중 하나는 보장할 수 없음.
- 지연 시간을 예측할 수 없기 때문에 네트워크 지연인지, 실패인지 알 수가 없으므로.
- [[FLP Theorem|FLP]] 불가능성 이론
- [[Two Generals' Problem]]