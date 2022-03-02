# 19. Advanced Features - Advanced Functions and Closures

> Next, we’ll explore some advanced features related to functions and closures, which include function pointers and returning closures.
> 
- TOC

## Function Pointer

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!("The answer is: {}", answer);
}
```

## Returning Closures

```rust
fn returns_closure() -> dyn Fn(i32) -> i32 {
    |x| x + 1
}

/// error[E0746]: return type cannot have an unboxed trait object
```

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```