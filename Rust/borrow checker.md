- `variable`이 스코프를 벗어난다면 
- `borrow checker`가 `variable`에 저장된 `value` 값을 메모리에서 해제해야 하는지 확인, 해당 값이 다른 `variable`에 의해 사용될 수 있게 함

```rust
fn main() {
    // variable x is declared and initialized in the main function
    let x = 5;

    // variable x is in scope within the curly braces of the main function
    println!("The value of x is: {}", x);

    {
        // variable y is declared and initialized within a new block of code
        let y = 10;

        // variable y is in scope within the curly braces of this block
        println!("The value of y is: {}", y);
    }

    // variable y is no longer in scope after the curly braces end
    // the following line will cause a compile-time error because y is not in scope
    // println!("The value of y is: {}", y);
}
```

main 함수 내부에서 `{}`를 벗어난 이후 y는 스코프 바깥에서 참조될 수 없습니다.