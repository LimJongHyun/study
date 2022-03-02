# 5. Using Structs to Structure Related Data

A *struct*, or *structure*, **is a custom data type that lets you name** and package together multiple related values that make up a meaningful group. If you’re familiar with an object-oriented language, a *struct*is like an object’s data attributes. In this chapter, we’ll compare and contrast tuples with structs. We’ll demonstrate how to define and instantiate structs. We’ll discuss how to define associated functions, especially the kind of associated functions called *methods*, to specify behavior associated with a struct type. Structs and enums (discussed in Chapter 6) are the **building blocks for creating new types in your program’s domain to take full advantage of Rust’s compile time type checking**.

---

**Defining and Instantiating Structs**

```rust
struct User {
    **active: bool, // << Field**
    username: String,
    email: String,
    sign_in_count: u64,
}
```

```rust
// Immutable
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
~~user1.username = String::from("sometwo@example.com");~~

// Mutable
let mut user1 = User {
		email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
user1.username = String::from("sometwo@example.com");
```

Note that the entire instance must be mutable; Rust doesn’t allow us to mark only certain fields as mutable.

**Using the Field Init Shorthand when Variables and Fields Have the Same Name**

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}

fn build_user(**email**: String, **username**: String) -> User {
    User {
        **email,
        username,**
        active: true,
        sign_in_count: 1,
    }
}
```

**Defining Methods**

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

Rust doesn’t have an equivalent to the → operator; instead, Rust has a feature called *automatic referencing and dereferencing*. Calling methods is one of the few places in Rust that has this behavior.

**Associated Functions**

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
}
```

**Multiple impl Blocks**

```rust
impl Rectangle {
    fn area(&self /* self: &Self */) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

**Unit-Like Structs Without Any Fields**

```rust
struct AlwaysEqual; // << Unit-like struct
let subject = AlwaysEqual;
```

**Refactoring with Tuples**

As-is

```rust
fn main() {
    let rect1 = (30, 50);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

To-be

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

**Adding Useful Functionality with Derived Traits**

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {:?}", rect1); // rect1 is Rectangle { width: 30, height: 50 }
}
```

**Representations**

All user-defined composite types (`struct`s, `enum`s, and `union`s) have a *representation* that specifies what the layout is for the type. The possible representations for a type are:

- [Default](https://doc.rust-lang.org/reference/type-layout.html#the-default-representation) : no guarantees of data layout made by this representation.
- `[C](https://doc.rust-lang.org/reference/type-layout.html#the-c-representation)`
- The [primitive representations](https://doc.rust-lang.org/reference/type-layout.html#primitive-representations)
- `[transparent](https://doc.rust-lang.org/reference/type-layout.html#the-transparent-representation)`

```rust
#[repr(C)]
struct ThreeInts {
    first: i16,
    second: i8,
    third: i32
}

// Default representation, alignment lowered to 2.
#[repr(packed(2))]
struct PackedStruct {
    first: i16,
    second: i8,
    third: i32
}

// C representation, alignment raised to 8
#[repr(C, align(8))]
struct AlignedStruct {
    first: i16,
    second: i8,
    third: i32
}
```