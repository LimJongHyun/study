# Closures

> Closures: Anonymous Functions that Can Capture Their Environment
> 

## Closure Type Inference and Annotation

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

```rust
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5); // error[E0308]: mismatched types
```

## Storing Closures Using Generic Parameters and the Fn Traits

```rust
use std::thread;
use std::time::Duration;

struct Cacher<T>
where
    T: Fn(u32) -> u32,
{
    calculation: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
where
    T: Fn(u32) -> u32,
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            }
        }
    }
}

fn generate_workout(intensity: u32, random_number: u32) {
    let mut expensive_result = Cacher::new(|num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    });

    if intensity < 25 {
        println!("Today, do {} pushups!", expensive_result.value(intensity));
        println!("Next, do {} situps!", expensive_result.value(intensity));
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_result.value(intensity)
            );
        }
    }
}

fn main() {
    let simulated_user_specified_value = 10;
    let simulated_random_number = 7;

    generate_workout(simulated_user_specified_value, simulated_random_number);
}
```

## Limitations of the Cacher Implementation

```rust
struct Cacher <T, K, V>
     where T: Fn(K) -> V
{
    calculation: T,
    values: HashMap<K, V>,
}

impl<T, K, V> Cacher<T, K, V>
where
    T: Fn(&K) -> V,
    K: Hash + Eq + Clone,
    V: Copy
{
    fn new(calculation: T) -> Cacher<T, K, V> {
        Cacher {
            calculation,
            values: HashMap::new()
        }
    }

    fn value(&mut self, arg: K) -> V {
        match self.values.get(&arg) {
            Some(v) => *v,
            None => {
                let v = (self.calculation)(&arg);
                self.values.insert(arg, v);
                v
            },
        }
    }
}
```

## Capturing the Environment with Closures

[](https://medium.com/swlh/understanding-closures-in-rust-21f286ed1759)

![https://miro.medium.com/max/700/0*mmn1xMTh4_1Rm80d.jpg](https://miro.medium.com/max/700/0*mmn1xMTh4_1Rm80d.jpg)

- Closures are a combination of a function pointer (`fn`) and a context.
- A closure with no context is just a function pointer.
- A closure which has an immutable context belongs to `Fn`.
- A closure which has a mutable context belongs to `FnMut`.
- A closure that owns its context belongs to `FnOnce`.

```rust
struct MyStruct {
    text: &'static str,
    number: u32,
}
impl MyStruct {
    fn new (text: &'static str, number: u32) -> MyStruct {
        MyStruct {
            text: text,
            number: number,
        }
    }
    // We have to specify that 'self' is an argument.
    fn get_number (&self) -> u32 {
        self.number
    }
    // We can specify different kinds of ownership and mutability of self.
    fn inc_number (&mut self) {
        self.number += 1;
    }
    // There are three different types of 'self'
    fn destructor (self) {
        println!("Destructing {}", self.text);
    }
}
```

### fn

```rust
let obj1 = MyStruct::new("Hello", 15);
let obj2 = MyStruct::new("More Text", 10);
let closure1 = |x: &MyStruct| x.get_number() + 3;
assert_eq!(closure1(&obj1), 18);
assert_eq!(closure1(&obj2), 13);
```

### Fn

```rust
let obj1 = MyStruct::new("Hello", 15);
let obj2 = MyStruct::new("More Text", 10);
// obj1 is borrowed by the closure immutably.
let closure2 = |x: &MyStruct| x.get_number() + obj1.get_number();
assert_eq!(closure2(&obj2), 25);
// We can borrow obj1 again immutably...
assert_eq!(obj1.get_number(), 15);
// But we can't borrow it mutably.
// obj1.inc_number();               // ERROR
```

### FnMut

```rust
let mut obj1 = MyStruct::new("Hello", 15);
let obj2 = MyStruct::new("More Text", 10);
// obj1 is borrowed by the closure mutably.
let mut closure3 = |x: &MyStruct| {
    obj1.inc_number();
    x.get_number() + obj1.get_number()
};
assert_eq!(closure3(&obj2), 26);
assert_eq!(closure3(&obj2), 27);
assert_eq!(closure3(&obj2), 28);
// We can't borrow obj1 mutably or immutably
// assert_eq!(obj1.get_number(), 18);   // ERROR
// obj1.inc_number();                   // ERROR
```

### FnOnce

```rust
let obj1 = MyStruct::new("Hello", 15);
let obj2 = MyStruct::new("More Text", 10);
// obj1 is owned by the closure
let closure4 = |x: &MyStruct| {
    obj1.destructor();
    x.get_number()
};
```