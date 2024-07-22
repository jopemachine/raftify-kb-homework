### Election timeout

<hr>

팔로워가 일정 시간 동안 [heartbeat](./heartbeat.md)을 받지 못해 일어나는 것으로, 이 시간은 무작위로 선정된다. `Election timeout`이 일어나면 자신은 후보자가 된다.

이 시간이 무작위로 선정되는 이유는 [Split Vote](./Split Vote.md) 문제를 해결하기 위해서이다.