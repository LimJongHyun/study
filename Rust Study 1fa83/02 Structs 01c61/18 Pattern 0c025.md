# 18. Patterns and Matching

A pattern consists of some combination of the following:

- Literals
- Destructured arrays, enums, structs, or tuples
- Variables
- Wildcards
- Placeholders

This chapter is a reference on all things related to patterns. We’ll cover the valid places to use patterns, the difference between refutable and irrefutable patterns, and the different kinds of pattern syntax that you might see. By the end of the chapter, you’ll know how to use patterns to express many concepts in a clear way.

# [18.1 All the Places Patterns Can Be Used](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#all-the-places-patterns-can-be-used)

`[match` Arms](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#match-arms)

```rust
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

[Conditional `if let` Expressions](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#conditional-if-let-expressions)

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {}, as the background", color);
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

we can’t combine these two conditions into `if let Ok(age) = age && age > 30`. The shadowed `age` we want to compare to 30 isn’t valid until the new scope starts with the curly bracket.

The downside of using `if let` expressions is that the compiler doesn’t check exhaustiveness, whereas with `match` expressions it does. If we omitted the last `else` block and therefore missed handling some cases, the compiler would not alert us to the possible logic bug.

`[while let` Conditional Loops](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#while-let-conditional-loops)

```rust
let mut stack = Vec::new();

stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
  println!("{}", top);
}

// 3
// 2
// 1
```

`[for` Loops](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#for-loops)

In a `for` loop, the pattern is the value that directly follows the keyword `for`, so in `for x in y` the `x` is the pattern.

```rust
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}

// a is at index 0
// b is at index 1
// c is at index 2
```

`[let` Statements](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#let-statements)

```rust
let x = 5;
let PATTERN = EXPRESSION;
let (x, y, z) = (1, 2, 3);
```

```rust
let (x, y) = (1, 2, 3);

$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0308]: mismatched types
 --> src/main.rs:2:9
  |
2 |     let (x, y) = (1, 2, 3);
  |         ^^^^^^   --------- this expression has type `({integer}, {integer}, {integer})`
  |         |
  |         expected a tuple with 3 elements, found one with 2 elements
  |
  = note: expected tuple `({integer}, {integer}, {integer})`
             found tuple `(_, _)`

For more information about this error, try `rustc --explain E0308`.
error: could not compile `patterns` due to previous error
```

[Function Parameters](https://doc.rust-lang.org/book/ch18-01-all-the-places-for-patterns.html#function-parameters)

```rust
fn foo(x: i32) {
    // code goes here
}

fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

We can also use patterns in closure parameter lists in the same way as in function parameter lists, because closures are similar to functions, as discussed in Chapter 13.

At this point, you’ve seen several ways of using patterns, but patterns don’t work the same in every place we can use them. In some places, the patterns must be **irrefutable**; in other circumstances, they can be **refutable**. We’ll discuss these two concepts next.

**irrefutable** : 반박 불가 - 어떤값이든 대응 해야한다.

**refutable** : 반박 가능

# [18.2 Refutability: Whether a Pattern Might Fail to Match](https://doc.rust-lang.org/book/ch18-02-refutability.html#refutability-whether-a-pattern-might-fail-to-match)

**Patterns come in two forms: refutable and irrefutable.**

Patterns that will match for any possible value passed are *irrefutable*

An example would be `x` in the statement `let x = 5;`

Patterns that can fail to match for some possible value are *refutable*.

An example would be `Some(x)` in the expression `if let Some(x) = a_value` because if the value in the `a_value` variable is `None` rather than `Some`, the `Some(x)` pattern will not match.

```rust
let Some(x) = some_option_value;

------------------------------------------------------------
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0005]: refutable pattern in local binding: `None` not covered
   --> src/main.rs:3:9
    |
3   |     let Some(x) = some_option_value;
    |         ^^^^^^^ pattern `None` not covered
    |
    = note: `let` bindings require an "irrefutable pattern", like a `struct` or an `enum` with only one variant
    = note: for more information, visit https://doc.rust-lang.org/book/ch18-02-refutability.html
    = note: the matched value is of type `Option<i32>`
help: you might want to use `if let` to ignore the variant that isn't matched
    |
3   |     if let Some(x) = some_option_value { /* */ }
    |

For more information about this error, try `rustc --explain E0005`.
error: could not compile `patterns` due to previous error
```

```rust
if let Some(x) = some_option_value {
    println!("{}", x);
}
```

```rust
if let x = 5 {
    println!("{}", x);
};

------------------------------------------------------------
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
warning: irrefutable `if let` pattern
 --> src/main.rs:2:5
  |
2 | /     if let x = 5 {
3 | |         println!("{}", x);
4 | |     };
  | |_____^
  |
  = note: `#[warn(irrefutable_let_patterns)]` on by default
  = note: this pattern will always match, so the `if let` is useless
  = help: consider replacing the `if let` with a `let`

warning: `patterns` (bin "patterns") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.39s
     Running `target/debug/patterns`
5
```

For this reason, match arms must use refutable patterns, except for the last arm, which should match any remaining values with an irrefutable pattern. Rust allows us to use an irrefutable pattern in a `match` with only one arm, but this syntax isn’t particularly useful and could be replaced with a simpler `let` statement.

# [18.3 Pattern Syntax](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#pattern-syntax)

[Matching Literals](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#matching-literals)

```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

[Matching Named Variables](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#matching-named-variables)

Named variables are irrefutable patterns that match any value, and we’ve used them many times in the book. However, there is a complication when you use named variables in `match` expressions. Because `match` starts a new scope, variables declared as part of a pattern inside the `match` expression will shadow those with the same name outside the `match` construct, as is the case with all variables.

```rust
let x = Some(5);
let y = 10;

match x {
    Some(50) => println!("Got 50"),
    Some(y) => println!("Matched, y = {:?}", y),
    _ => println!("Default case, x = {:?}", x),
}

println!("at the end: x = {:?}, y = {:?}", x, y);

//Matched, y = 5
//at the end: x = Some(5), y = 10
```

To create a `match` expression that compares the values of the outer `x` and `y`, rather than introducing a shadowed variable, we would need to use a match guard conditional instead. We’ll talk about match guards later in the [“Extra Conditionals with Match Guards”](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#extra-conditionals-with-match-guards) section.

[Multiple Patterns](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#multiple-patterns)

```rust
let x = 1;

match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

[Matching Ranges of Values with `..=`](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#matching-ranges-of-values-with-)

```rust
let x = 5;

match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
}
// one through five

match x {
    1 | 2 | 3 | 4 | 5 => println!("one through five"),
    _ => println!("something else"),
}
// one through five

match x {
    x if 1 <= x && x <=5 => println!("one through five"),
    _ => println!("something else"),
}

// one through five

match x {
    x if 1 <= x && x <=5 => println!("one through five"),
    x if 5 <= x && x <=6 => println!("five through six"),
    _ => println!("something else"),
}

// one through five

// lex x = 6;
// five through six
```

```rust
let x = 'c';

match x {
  'a'..='j' => println!("early ASCII letter"),
  'k'..='z' => println!("late ASCII letter"),
  _ => println!("something else"),
}
```

[Destructuring to Break Apart Values](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-to-break-apart-values)

[Destructuring Structs](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-structs)

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

```rust
fn main() {

    let p = Point { x: 0, y: 7 };

    let Point { x, a } = p;
    assert_eq!(0, x);
    assert_eq!(7, a);
}

--------------------------------------------------

PS C:\Works\rust\hello-rust> cargo run
   Compiling hello-rust v0.1.0 (C:\Works\rust\hello-rust)
error[E0026]: struct `Point` does not have a field named `a`
  --> src\main.rs:10:20
   |
10 |     let Point { x, a } = p;
   |                    ^
   |                    |
   |                    struct `Point` does not have this field
   |                    help: a field with a similar name exists: `y`

For more information about this error, try `rustc --explain E0026`.
error: could not compile `hello-rust` due to previous error
```

```rust
fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
}

// On the y axis at 7
```

[Destructuring Enums](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-enums)

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.")
        }
        Message::Move { x, y } => {
            println!(
                "Move in the x direction {} and in the y direction {}",
                x, y
            );
        }
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => println!(
            "Change the color to red {}, green {}, and blue {}",
            r, g, b
        ),
    }
}
```

[Destructuring Nested Structs and Enums](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-nested-structs-and-enums)

```rust
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => println!(
            "Change the color to red {}, green {}, and blue {}",
            r, g, b
        ),
        Message::ChangeColor(Color::Hsv(h, s, v)) => println!(
            "Change the color to hue {}, saturation {}, and value {}",
            h, s, v
        ),
        _ => (),
    }
}
```

[Destructuring Structs and Tuples](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#destructuring-structs-and-tuples)

```rust
let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
```

[Ignoring Values in a Pattern](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-values-in-a-pattern)

[Ignoring an Entire Value with `_`](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-an-entire-value-with-_)

```rust
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}

fn main() {
    foo(3, 4);
}
```

[Ignoring Parts of a Value with a Nested `_`](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-parts-of-a-value-with-a-nested-_)

```rust
let mut setting_value = Some(5);
let new_setting_value = Some(10);

match (setting_value, new_setting_value) {
    (Some(_), Some(_)) => {
        println!("Can't overwrite an existing customized value");
    }
    _ => {
        setting_value = new_setting_value;
    }
}

println!("setting is {:?}", setting_value);

// Can't overwrite an existing customized value
```

```rust
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, _, third, _, fifth) => {
        println!("Some numbers: {}, {}, {}", first, third, fifth)
    }
}

// Some numbers: 2, 8, 32
```

[Ignoring an Unused Variable by Starting Its Name with `_`](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-an-unused-variable-by-starting-its-name-with-_)

```rust
fn main() {
    
    let x = 5;
    let y = 10;

    println!("{}", y);
}

----
PS C:\Works\rust\hello-rust> cargo run
   Compiling hello-rust v0.1.0 (C:\Works\rust\hello-rust)
warning: unused variable: `x`
 --> src\main.rs:3:9
  |
3 |     let x = 5;
  |         ^ help: if this is intentional, prefix it with an underscore: `_x`
  |
  = note: `#[warn(unused_variables)]` on by default

warning: `hello-rust` (bin "hello-rust") generated 1 warning 
    Finished dev [unoptimized + debuginfo] target(s) in 0.46s
     Running `target\debug\hello-rust.exe`
10
```

```rust
fn main() {
    
    let _x = 5;
    let y = 10;

    println!("{}", y);
}

----
PS C:\Works\rust\hello-rust> cargo run
   Compiling hello-rust v0.1.0 (C:\Works\rust\hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.41s
     Running `target\debug\hello-rust.exe`
10
```

```rust
let s = Some(String::from("Hello!"));

if let Some(_s) = s {
    println!("found a string");
}

println!("{:?}", s);

----
PS C:\Works\rust\hello-rust> cargo run
   Compiling hello-rust v0.1.0 (C:\Works\rust\hello-rust)
error[E0382]: borrow of partially moved value: `s`
 --> src\main.rs:8:22
  |
4 |     if let Some(_s) = s {
  |                 -- value partially moved here
...
8 |     println!("{:?}", s);
  |                      ^ value borrowed here after partial move
  |
  = note: partial move occurs because value has type `String`, which does not implement the `Copy` trait
help: borrow this field in the pattern to avoid moving `s.0`
  |
4 |     if let Some(ref _s) = s {
  |                 +++

For more information about this error, try `rustc --explain E0382`.
error: could not compile `hello-rust` due to previous error
```

```rust
fn main() {
    let s = Some(String::from("Hello!"));

    if let Some(_) = s {
        println!("found a string");
    }

    println!("{:?}", s);
}

----
PS C:\Works\rust\hello-rust> cargo run
   Compiling hello-rust v0.1.0 (C:\Works\rust\hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running `target\debug\hello-rust.exe`
found a string
Some("Hello!")
```

[Ignoring Remaining Parts of a Value with `..`](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#ignoring-remaining-parts-of-a-value-with-)

```rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}
```

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {}, {}", first, last);
        }
    }
}

// Some numbers: 2, 32
```

However, using `..` must be unambiguous. If it is unclear which values are intended for matching and which should be ignored, Rust will give us an error. Listing 18-25 shows an example of using `..` ambiguously, so it will not compile.

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (.., second, ..) => {
            println!("Some numbers: {}", second)
        },
    }
}

----

PS C:\Works\rust\hello-rust> cargo run
   Compiling hello-rust v0.1.0 (C:\Works\rust\hello-rust)
error: `..` can only be used once per tuple pattern
 --> src\main.rs:5:22
  |
5 |         (.., second, ..) => {
  |          --          ^^ can only be used once per tuple pattern
  |          |
  |          previously used here

error: could not compile `hello-rust` due to previous error
```

[Extra Conditionals with Match Guards](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#extra-conditionals-with-match-guards)

```rust
let num = Some(4);

match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}

// less than five: 4
```

[이전 문제](18%20Pattern%200c025.md)

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {:?}", n),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {:?}", x, y);
}

// Default case, x = Some(5)
// at the end: x = Some(5), y = 10
```

```rust
let x = 4;
let y = false;

match x {
    4 | 5 | 6 if y => println!("yes"),
    _ => println!("no"),
}

// no
// (4 | 5 | 6) if y : O
// 4 | 5 | (6 if y) : X
```

`[@` Bindings](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#-bindings)

The *at* operator (`@`) lets us create a variable that holds a value at the same time we’re testing that value to see whether it matches a pattern.

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello {
        id: id_variable @ 3..=7,
    } => println!("Found an id in range: {}", id_variable),
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    }
    Message::Hello { id } => println!("Found some other id: {}", id),
}

// Found an id in range: 5
```

we’re capturing whatever value matched the range while also testing that the value matched the range pattern.