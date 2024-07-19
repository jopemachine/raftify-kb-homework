---- 
##### CAP 셋 다 모두 만족하는 분산 시스템은 있을 수 는 없다.

###### C : [[Consistency]]
###### A : [[Availability]]
###### P : [[Partition Tolerance]]

여기서 P를 만족하지 못하면, 분산 시스템 자체가 불가능.
고로 우리는 극단적인 상황에서, [[Consistency|C]]와 [[Availability|A]] 중 하나를 포기해야 함.

- C 포기 -> [[CP System]]
- A 포기 -> [[AP System]]
![[Pasted image 20240720031709.png]]