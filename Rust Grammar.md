## Pattern Matching

### `if let` 표현식

```rust
if let PATTERN = EXPRESSION {
    // 주어진 PATTERN 이 EXPRESSION 과 매칭되면 여기를 실행함.
}
```

## Trait

- 어떤 타입이 가진 기능을 정의함. 다른 타입과 공유할 수 있음. 다른 언어에서의 인터페이스와 비슷함 (하지만 조금 다르다)

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

- `Summary`: trait 의 이름. (오~ trait 한테도 이름을 붙여줘? 생각해보니 자바도 똑같음.ㅋㅋ)
- 위와 같이 적어놓으면 Summary trait 의 구현체는 반드시 `summarize` 함수에 대한 body 를 제공해야 함.
  - 그게 아니라 trait 정의할 때 body까지 같이 정의했다면 그게 default 동작이 됨.

### Implement Trait

- 자바와 다른 점: 클래스와 trait 는 독립적임. 하나의 trait 을 "특정 클래스에 대해" implement 할 수 있음. (오~~~~ 좋은데)

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

- 이렇게 하면 개발자는 NewArticle, Tweet 에 대한 `summarize` 함수를 호출할 수 있음.
  - 오~~~~ 좋다. 특정 타입별로 동작을 정의하는게 아니라 공통된 동작을 특정 타입에 대해서 정의함.

### Trait 사용하기

#### 파라미터로 넘길 수 있음. 오~

```rust
// 이렇게 하면 Summary 를 implement 한 구현체라면 무엇이든 item 자리에 올 수 있다.
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

// Syntatic sugar
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
// Syntatic sugar 는 이럴 때 유용하다.
pub fn notify<T: Summary>(item1: &T, item2: &T) { // T 가 이거처럼 여러번 쓰일 때.
```

```rust
// trait 끼리 합칠 수도 있다.
// 이렇게 하면 item 은 Summary 와 Display 모두를 구현해야 함.
pub fn notify(item: &(impl Summary + Display)) {
  
// Syntatic sugar
pub fn notify<T: Summary + Display>(item: &T) {
```

```rust
// 너무 많은 trait 때문에 파라미터 칸이 길어지는게 보기 싫을때를 위한 `where` 절

// as-is:
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {

// to-be:
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
```

#### 리턴값으로도 넘길 수 있다. 오~ 단, trait 자체를 리턴하는게 아니라 그걸 구현하는 구현체를 리턴해야 함.

```rust
fn returns_summarizable() -> impl Summary {
    // 하나만 리턴할 수 있음. Summary 를 Tweet, NewsArticle 2개에 대해 구현했다고 하더라도 둘 다 리턴할수는 없다. (당연함. 안그러면 함수가 아님.)
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```

## Macros

러스트에서 매크로라고 하면 총 4가지 종류가 있다. Declarative 매크로 1개, procedural 매크로 3개.

- Declarative Macro: `macro_rules!` 생성자를 써서 정의함. 컴파일 시점에 주어진 expression을 확인하고 그 패턴과 매칭되는 동작을 하도록 코드를 '바꿔치기'해서 컴파일 함.

  - ```rust
    #[macro_export] // 그래야 crate 내부 다른 곳에서도 쓸 수 있음.
    macro_rules! vec { // 매크로 이름 정의.
        ( $( $x:expr ),* ) => { // 이 패턴과 매칭되면 body 를 실행한다. 다른 애가 들어오면 에러가 남!
            {
                let mut temp_vec = Vec::new();
                $(
                    temp_vec.push($x);
                )*
                temp_vec
            }
        };
    }
    ```

  - 예) `println!`
  - “macros by example,” “`macro_rules!` macros,” or just plain “macros.” 라고도 불린다.

- 나머지는 Procedural Macro. <--> Declarative Macro. 특정 코드를 input으로 넣으면 그 코드를 실행하고, output으로 또 어떤 코드를 만들어낸다.

  - ```rust
    use proc_macro; // TokenStream 이 이 crate 안에 있음.
    
    #[some_attribute]
    pub fn some_name(input: TokenStream) -> TokenStream {
    }
    ```

    - `TokenStream`: 절차적 매크로의 핵심. input, output (코드 조각) 모두 이 타입임.

  - Custom Macro: `#[drive]`:- struct 나 enum 의 `derive` 속성으로 특정 코드를 추가할 수 있음.

    - 이렇게 하는 이유: 컴파일 시점에 코드를 바꿔치기 하려고.

    ```rust
    // read_only.rs
    
    #[derive(Default, Debug, PartialEq, Eq, Clone)] // 이 매크로 덕분에 ReadState 에 대해 Default, Debug, PartialEq, Eq, Clone 에 있는 메소드를 사용할 수 있음. 이야~~~~
    pub struct ReadState {
        /// The index of the read state.
        pub index: u64,
        /// A datagram consisting of context about the request.
        pub request_ctx: Vec<u8>,
    }
    ```

  - Attribute-like Macro: 어디든지(?) 커스텀 속성을 정의할 수 있는 매크로

    - 코드를 만들어내는 `derive` 와 다르게 attribute-like macro 는 새 attribute 를 만들 수 있으며 좀 더 유연함 - 구조체나 enum뿐만 아니라 함수 같은 다른 러스트 아이템에도 적용할 수 있다.

    ```rust
    #[route(GET, "/")] // 이렇게 써먹고
    fn index() { // ...
      
    #[proc_macro_attribute] // 매크로는 이렇게 정의한다
    pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
    ```

  - Function-like Macro: 함수 호출처럼 보이지만 인자로 넘어간 토큰에 대해서 동작하는 매크로

    - 함수보다 유연함. Unknown 개수의 인자를 받을 수 있음.

      ```rust
      let sql = sql!(SELECT * FROM posts WHERE id=1); // 이렇게 쓰고
      
      #[proc_macro] // 이렇게 정의한다.
      pub fn sql(input: TokenStream) -> TokenStream { // ...
      ```


# Macros

## Tokio::select!

- 주어진 브랜치의 작업이 끝나기까지 기다리고, 먼저 작업이 끝난 브랜치의 블록을 실행함.
  - 일종의 콜백함수 등록 이구만

```rust
tokio::select! {
    // Branch 1
    some_variable = some_async_operation => {
        // Code to execute if this operation completes first
    },
    // Branch 2
    other_variable = another_async_operation => {
        // Code to execute if this operation completes first
    },
    // Additional branches can be added as needed
}
```

