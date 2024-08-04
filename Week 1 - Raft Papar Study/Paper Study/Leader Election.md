[[heartbeat]] 라는 메커니즘을 통해 리더를 선출.

[[term]] 마다 하나의 리더만 선출됨. Election Safety

리더가 `Election timeout`이라고 부르는 정해진 기간 동안 RPC를 보내지 않으면, 리더에 장애가 생긴 것으로 간주하고 새로운 리더 선출 프로세스를 시작


