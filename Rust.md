# Result

- Result: Ok(T) 혹은 Err(E) 타입이 될 수 있음.
- `unwrap()` 으로 안에 있는 결과값을 꺼낼 수 있음
  - Ok(T) 라면: T 를 리턴
  - Err(E) 라면: 프로그램 panic발생, 에러 메세지 노출

# Ownership

- Rust 의 오너십 모델은 GC 없이도 메모리를 안전하게 사용할 수 있도로 고안되었다.
- 오너십 모델에서는 메모리 할당, 접근, 해제를 관리한다.

### Ownersip

- 러스트의 모든 value 는 하나의 owner 를 가진다. Owner 는 그 값이 더이상 필요없어졌을 때 제거하는 역할을 담당한다. (예: 변수가 할당된 scope 이 끝났을 때)
- 만약 owner 가 scope 를 벗어났다면 그 값 역시 자동으로 메모리에서 해제된다.

### Borrowing

- 러스트에서는 값을 reference 로 참조하는 것을 허용한다. (`&T`: immutbale, `&mut T`: mutable)
- Borrowing 을 사용하면 값의 오너십을 바꾸지 않고 그 값을 사용할 수 있게 된다. 단, race condition 을 막기 위해 다음 제약 조건이 붙는다.
  - 사용자는 언제든 하나의 값에 대해 immutable 참조를 여러개 가지거나, 딱 1개의 mutable 참조를 가질 수 있다.
    - Mutable 참조가 여러개면 안되니까.

### Move Semantics

- 값을 새 변수에 할당하거나 함수에 넘겨줄 때, 오너십이 그 새로운 변수나 함수로 이동한다. (바뀐다.)
- 한 번 바뀐 다음에는 기존 변수는 더이상 유효하지 않게 된다.
  - 이는 중복으로 memory free 를 하거나 dangling pointer 가 발생하는 것을 막기 위함.

### 예시

```rust
fn main() {
    let s = String::from("hello"); // `s` owns the string
    let s1 = s; // `s1` now owns the string, and `s` is no longer valid
    println!("{}", s); // 컴파일 에러 !! 여기서 s 라는 변수는 사용될 수 없다. s 라는 변수의 값이 s1에 할당됨과 동시에, 그 값에 대한 소유권은 s1 으로 넘어가고 s 는 더이상 유효하지 않은 변수가 되어버렸기 때문.

    let s2 = s1.clone(); // 대신 이렇게 clone 해주면
    println!("s1: {}, s2: {}", s1, s2); // s1은 여전히 유효하다.
}
```

# Task

- Task 는 러스트에서 _동시에_ 실행될 수 있는 경량 비동기 실행 단위다.
- 스레드와 비슷하지만, 논블로킹이고 Tokio 와 같은 비동기 런타임에 의해 관리된다는 점이 다르다.

### 비동기 작업과 오너십

- 비동기 코드를 실행할 때에도 러스트이 오너십 모델이 적용된다. 즉, 비동기 태스크로 넘어간 값이나 `.await` 를 통해 기다려서 받은 값은 반드시 위의 오너십 룰, borrowing rule을 따라야 한다.

### 예시

```rust
use tokio::task;

#[tokio::main]
async fn main() {
    let data = String::from("hello"); // data 에 String 값을 할당함

    // 새 비동기 태스크를 하나 만들었음. 그 안에서 data 를 사용하기 때문에 오너십은 해당 태스크가 갖게 된다.
    let handle = task::spawn(async move {
        println!("Data inside the task: {}", data);
    })ㅇ

    // 이 영역에서는 더이상 data 변수를 사용할 수 없음. 오너십이 main 에서, 위 move 로 넘어갔기 때문이다.

    handle.await.unwrap();
}
```

