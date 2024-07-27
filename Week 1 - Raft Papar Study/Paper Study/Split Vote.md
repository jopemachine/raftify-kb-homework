[[Leader Election]] 과정에서 복수개의 node는 동시에 Candidate이 될 수 있음.
이 경우 복수개의 node 각각이 [[RequestVote]] 메세지를 전송하게 됨.
이러한 현상을 Split vote라고 함