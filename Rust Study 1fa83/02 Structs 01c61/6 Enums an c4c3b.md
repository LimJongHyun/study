# 6. Enums and Pattern Matching

In this chapter we’ll look at *enumerations*, also referred to as *enums*. Enums allow you to define a type by enumerating its possible *variants*. First, we’ll define and use an enum to show how an enum can encode meaning along with data. Next, we’ll explore a particularly useful enum, called `Option`, which expresses that a value can be either something or nothing. Then we’ll look at how pattern matching in the `match` expression makes it easy to run different code for different values of an enum. Finally, we’ll cover how the `if let` construct is another convenient and concise idiom available to you to handle enums in your code.

Enums are a feature in many languages, but their capabilities differ in each language. Rust’s enums are most similar to *algebraic data types* in functional languages, such as F#, OCaml, and Haskell.

---

# [6.1 Defining an Enum](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#defining-an-enum)

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

## [Enum Values](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#enum-values)

```rust
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

```rust
fn route(ip_kind: IpAddrKind) {}
```

```rust
route(IpAddrKind::V4);
route(IpAddrKind::V6);

route(four);
route(six);
```

```rust
  enum IpAddrKind {
      V4,
      V6,
  }

  struct IpAddr {
      kind: IpAddrKind,
      address: String,
  }

  let home = IpAddr {
      kind: IpAddrKind::V4,
      address: String::from("127.0.0.1"),
  };

  let loopback = IpAddr {
      kind: IpAddrKind::V6,
      address: String::from("::1"),
};
```

```rust
enum IpAddr {
    V4(String),
    V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));

let loopback = IpAddr::V6(String::from("::1"));
```

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```

standard library

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

[https://doc.rust-lang.org/std/net/enum.IpAddr.html](https://doc.rust-lang.org/std/net/enum.IpAddr.html)

```rust
use std::net::{IpAddr, Ipv4Addr, Ipv6Addr};

let localhost_v4 = IpAddr::V4(Ipv4Addr::new(127, 0, 0, 1));
let localhost_v6 = IpAddr::V6(Ipv6Addr::new(0, 0, 0, 0, 0, 0, 0, 1));

assert_eq!("127.0.0.1".parse(), Ok(localhost_v4));
assert_eq!("::1".parse(), Ok(localhost_v6));

assert_eq!(localhost_v4.is_ipv6(), false);
assert_eq!(localhost_v4.is_ipv4(), true);
```

```rust
#[derive(Debug)]
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        // method body would be defined here
        match self {
            Message::Write(write) => println!("{:?} : {}", self, write),
            Message::Quit => println!("{:?}", self),
            Message::ChangeColor(a,b,c) => println!("{:?} : {}, {}, {}", self, a, b, c),
            Message::Move { x, y } => println!("{:?} : {}, {}", self, x, y),
        }
    }
}

fn main() {

    let w = Message::Write(String::from("hello"));
    w.call();   //Write("hello") : hello

    let q = Message::Quit;
    q.call();   //Quit

    let c = Message::ChangeColor( 1, 2, 3 );
    c.call();   //ChangeColor(1, 2, 3) : 1, 2, 3

    let m = Message::Move { x: 4, y: 5 };
    m.call();   //Move { x: 4, y: 5 } : 4, 5
}
```

vs

```rust
struct QuitMessage; // unit struct
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // tuple struct
struct ChangeColorMessage(i32, i32, i32); // tuple struct
```

## [The `Option` Enum and Its Advantages Over Null Values](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#the-option-enum-and-its-advantages-over-null-values)

This enum is `Option<T>`, and it is [defined by the standard library](https://doc.rust-lang.org/std/option/enum.Option.html) as follows:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

```rust
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
```

`Option<T>` and `T` (where `T` can be any type) are different types, the compiler won’t let us use an `Option<T>` value as if it were definitely a valid value.

```rust
let x: i8 = 5;
let y: Option<i8> = Some(5);

let sum = x + y;
```

```
$ cargo run
   Compiling enums v0.1.0 (file:///projects/enums)
error[E0277]: cannot add `Option<i8>` to `i8`
 --> src/main.rs:5:17
  |
5 |     let sum = x + y;
  |                 ^ **no implementation for `i8 + Option<i8>`**
  |
  = help: the trait `Add<Option<i8>>` is not implemented for `i8`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `enums` due to previous error
```

you have to convert an `Option<T>` to a `T` before you can perform `T` operations with it. Generally, this helps catch one of the most common issues with null: assuming that something isn’t null when it actually is.

[https://doc.rust-lang.org/std/option/enum.Option.html](https://doc.rust-lang.org/std/option/enum.Option.html)

---

# [6.2 The `match` Control Flow Operator](https://doc.rust-lang.org/book/ch06-02-match.html#the-match-control-flow-operator)

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

## [Patterns that Bind to Values](https://doc.rust-lang.org/book/ch06-02-match.html#patterns-that-bind-to-values)

From 1999 through 2008, the United States minted quarters with different designs for each of the 50 states on one side. No other coins got state designs, so **only quarters** have this extra value. We can add this information to our `enum` by changing the `Quarter` variant to include a `UsState` value stored inside it,

```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

## [Matching with `Option<T>`](https://doc.rust-lang.org/book/ch06-02-match.html#matching-with-optiont)

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

[Matches Are Exhaustive](https://doc.rust-lang.org/book/ch06-02-match.html#matches-are-exhaustive)

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
    }
}
```

```rust
$ cargo run
   Compiling enums v0.1.0 (file:///projects/enums)
error[E0004]: non-exhaustive patterns: `None` not covered
   --> src/main.rs:3:15
    |
3   |         match x {
    |               ^ **pattern `None` not covered**
    |
    = help: ensure that all possible cases are being handled, possibly by adding wildcards or more match arms
    = note: the matched value is of type `Option<i32>`
```

[Catch-all Patterns and the `_` Placeholder](https://doc.rust-lang.org/book/ch06-02-match.html#catch-all-patterns-and-the-_-placeholder)

```rust
let dice_roll = 9;
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    other => move_player(other),
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn move_player(num_spaces: u8) {}
```

```rust
let dice_roll = 9;
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    _ => reroll(),
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn reroll() {}
```

```rust
let dice_roll = 9;
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    _ => (), // empty tuple type
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
```

# [6.3 Concise Control Flow with `if let`](https://doc.rust-lang.org/book/ch06-03-if-let.html#concise-control-flow-with-if-let)

```rust
let config_max = Some(3u8); // Option<u8>
match config_max {
    Some(max) => println!("The maximum is configured to be {}", max),
    _ => (),
}
```

```rust
let config_max = Some(3u8); // Option<u8>
if let Some(max) = config_max {
    println!("The maximum is configured to be {}", max);
}
```

Using `if let` means less typing, less indentation, and less boilerplate code. However, you lose the exhaustive checking that `match` enforces. Choosing between `match` and `if let` depends on what you’re doing in your particular situation and whether gaining conciseness is an appropriate trade-off for losing exhaustive checking.

```rust
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}
```

```rust
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```