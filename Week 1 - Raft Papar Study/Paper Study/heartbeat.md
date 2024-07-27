어느 node가 Leader 역할을 맡게 되면, Follower에게 주기적으로 신호를 보냄
주기적으로 보내는 신호를 heartbeat이라고 함

만약 follower가 [[Election timeout]]안에 heartbeat 신호를 받지 못한다면 candidate 역할을 맡게되고, [[Leader Election]] 과정을 진행하게 된다