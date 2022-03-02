# 3.
Common Programming Concepts

## Variable and Mutability

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    // error[E0384]: cannot assign twice to immutable variable `x`
    x = 6;
    println!("The value of x is: {}", x);

    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);

    // Constants type of value must be annotated.
    const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;

    // Shadowing
    let x = 5;
    let x = x + 1;
    {
        // 12 = 6 * 2;
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }
    // x = 6
    println!("The value of x is: {}", x);
    // Shadowing different types
    let spaces = "   ";
    let spaces = spaces.len();
}
```

---

## Data types

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

### Scalar Types

Integer Types

| Length | Signed | Unsigned |
| --- | --- | --- |
| 8-bit | i8 | u8 |
| 16-bit | i16 | u16 |
| 32-bit | i32 | u32 |
| 64-bit | i64 | u64 |
| 128-bit | i128 | u128 |
| arch | isize | usize |

```rust
// explicit handle overflow
let x = u8::wrapping_add(255, 1);
let y = u8::saturating_add(255, 1);
let z = u8::checked_add(255, 1);
let w = u8::overflowing_add(255, 1);
println!("{}, {}, {:?}, {:?}", x, y, z, w);

// 0, 255, None, (0, true)
```

Integer Literals

| Number literals | Example |
| --- | --- |
| Decimal | 98_222 |
| Hex | 0xff |
| Octal | 0o77 |
| Binary | 0b1111_0000 |
| Byte (u8 only) | b'A' |

Floating-Point Types

```rust
let x = 2.0; // f64
let y: f32 = 3.0; // f32
```

Numeric Operations

```rust
// addition
let sum = 5 + 10;

// subtraction
let difference = 95.5 - 4.3;

// multiplication
let product = 4 * 30;

// division
let quotient = 56.7 / 32.2;
let floored = 2 / 3; // Results in 0

// remainder
let remainder = 43 % 5;
```

The Boolean Type

```rust
let t = true;
let f: bool = false; // with explicit type annotation
```

The Character Type

4 bytes in size, represents Unicode Scalar Value

```rust
let c = 'z';
let z = 'ℤ';
let heart_eyed_cat = '😻';
```

### Compound Types

The Tuple Type

```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);
let tup = (500, 6.4, 1);
let (x, y, z) = tup;

println!("The value of y is: {}", y);

let x: (i32, f64, u8) = (500, 6.4, 1);

let five_hundred = x.0;
let six_point_four = x.1;
let one = x.2;
```

The Array Type

```rust
let a = [1, 2, 3, 4, 5];
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
let a: [i32; 5] = [1, 2, 3, 4, 5];
let a = [3; 5]; // let a = [3, 3, 3, 3, 3];

let first = a[0];
let second = a[1];
```

---

## Functions

```rust
fn main() {
    println!("Hello, world!");

    another_function();
}

fn another_function() {
    println!("Another function.");
}

fn another_function(x: i32) {
    println!("The value of x is: {}", x);
}
```

### Function Bodies Contain Statements and Expressions

```rust
fn main() {
    let x = (let y = 6); // 'let y = 6' is a statement, doesn't return a value.
}
```

```rust
fn main() {
    let y = {
        let x = 3;
        x + 1
    }; // This is an expression.

    println!("The value of y is: {}", y);
}
```

### Functions with Return Values

```rust
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {}", x);
}

// return value
fn plus_one(x: i32) -> i32 {
    x + 1
}
// no return
fn plus_one(x: i32) -> i32 {
    x + 1;
}
```

---

## Comments

```rust
// So we’re doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what’s going on.
fn main() {
    // I’m feeling lucky today
    let lucky_number = 7; // I’m feeling lucky today
}
```

---

## Control Flow

### if Expressions

```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }

		// error[E0308]: mismatched types
		if number {
				println!("number was three");
		}
}
```

Using if in a let Statement

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {}", number);
}
```

### Repetition with Loops

```rust
fn main() {
    loop {
        println!("again!");
    }
}
```

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {}", count);
        let mut remaining = 10;

        loop {
            println!("remaining = {}", remaining);
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {}", count);
}
```

Returning Values from Loops

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}
```

Other loops

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

		// Conditional Loops with while
    while index < 5 {
        println!("the value is: {}", a[index]);

        index += 1;
    }

		// Looping Through a Collection with for
		for element in a {
        println!("the value is: {}", element);
    }

		for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```