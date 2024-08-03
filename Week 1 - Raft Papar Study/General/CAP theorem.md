---- 
##### CAP 셋 다 모두 만족하는 분산 시스템은 있을 수 는 없다.

###### C : [[Consistency.md|Consistency]]
###### A : [[Availability.md|Availabilty]]
###### P : [[Partition Tolerance.md|Partition Tolerance]]

여기서 P를 만족하지 못하면, 분산 시스템 자체가 불가능.
고로 우리는 극단적인 상황에서, [[Consistency.md|C]]와 [[Availability.md|A]] 중 하나를 포기해야 함.

- C 포기 -> [[CP System.md|CP System]]
- A 포기 -> [[AP System.md|AP System]]
![[img/week1/General/Pasted image 20240720031709.png]]