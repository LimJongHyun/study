# 8. Common Colletions

# 8. Common Collections

Rust’s standard library includes a number of very useful data structures called *collections*. Most other data types represent one specific value, but collections can contain multiple values. **Unlike the built-in array and tuple types, the data these collections point to is stored on the heap**, which means the amount of data does not need to be known at compile time and can grow or shrink as the program runs. Each kind of collection has different capabilities and costs, and choosing an appropriate one for your current situation is a skill you’ll develop over time. In this chapter, we’ll discuss three collections that are used very often in Rust programs:

## [Vector](https://doc.rust-lang.org/std/vec/struct.Vec.html)

**Creating a New Vector**

```rust
let v: Vec<i32> = Vec::new();
let v2 = vec![1, 2, 3];
```

**Updating & Reading a Vector**

```rust
let mut v = Vec::new();
v.push(5);
v.push(6);

let third: &i32 = &v[2];
println!("The third element is {}", third);

match v.get(2) {
   Some(third) => println!("The third element is {}", third),
   None => println!("There is no third element."),
}

//~ Panic!
let does_not_exist = &v[100];
let does_not_exist = v.get(100);
//~ Panic!
```

**Recall the rule that states you can’t have mutable and immutable references in the same scope.** 

```rust
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];
v.push(6);

println!("The first element is: {}", first);
```

```rust
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:5
  |
4 |     let first = &v[0];
  |                  - immutable borrow occurs here
5 | 
6 |     v.push(6);
  |     ^^^^^^^^^ mutable borrow occurs here
7 | 
8 |     println!("The first element is: {}", first);
  |                                          ----- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `collections` due to previous error
```

**Iterating over the Values in a Vector**

```rust
// Immutable
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}

// Mutable
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

**Using an Enum to Store Multiple Types**

```rust
enum SpreadsheetCell {
	Int(i32),
	Float(f64),
	Text(String),
}

let row = vec![
   SpreadsheetCell::Int(3),
   SpreadsheetCell::Text(String::from("blue")),
   SpreadsheetCell::Float(10.12),
];
```

**Memory block**

```
ptr      len  capacity
+--------+--------+--------+
| 0x0123 |      2 |      4 |
+--------+--------+--------+
            |
            v
Heap   
+--------+--------+--------+--------+
|    'a' |    'b' | uninit | uninit |
+--------+--------+--------+--------+
```

---

### [String](https://doc.rust-lang.org/stable/std/string/struct.String.html)

The `String` type, which is provided by Rust’s standard library rather than coded into the core language, is a growable, mutable, owned, UTF-8 encoded string type. When Rustaceans refer to “strings” in Rust, they usually mean the `String` and the string slice `&str` types, not just one of those types. Although this section is largely about `String`, both types are used heavily in Rust’s standard library, and both `String` and string slices are **[UTF-8](https://www.charset.org/utf-8) encoded**.

**Creating a New String**

```rust
let mut s = String::new();

let data = "initial contents";

let s = data.to_string();

// the method also works on a literal directly:
let s = "initial contents".to_string();

let s = String::from("initial contents");

let hello = String::from("السلام عليكم");
let hello = String::from("Dobrý den");
let hello = String::from("안녕하세요");
```

**Concatenation with the `+` Operator or the `format!` Macro**

```rust
fn add(self, s: &str) -> String {
```

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
```

```rust
Compiling playground v0.0.1 (/playground)
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:7:27
  |
3 |     let s1 = String::from("Hello, ");
  |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
4 |     let s2 = String::from("world!");
5 |     let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
  |              -- value moved here
6 |     
7 |      println!("s1 is {}", s1);
  |                           ^^ value borrowed here after move

For more information about this error, try `rustc --explain E0382`.
error: could not compile `playground` due to previous error
```

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3);

// Result : s is tic-tac-toe
```

**Indexing into Strings**

```rust
let s1 = String::from("hello");
let h = s1[0];

//------------------------------------------------------------------------------------
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0277]: the type `String` cannot be indexed by `{integer}`
 --> src/main.rs:3:13
  |
3 |     let h = s1[0];
  |             ^^^^^ `String` cannot be indexed by `{integer}`
  |
  = help: the trait `Index<{integer}>` is not implemented for `String`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `collections` due to previous error
```

The error and the note tell the story: **Rust strings don’t support indexing**. But why not? To answer that question, we need to discuss how Rust stores strings in memory.

**Internal Representation**

```rust
let hello = String::from("Hola"); // length : 4
let hello = String::from("Здравствуйте"); // length : 24
```

1. A `String` is a wrapper over a `Vec<u8>`
2. Encoded UTF-8
3. To avoid returning an unexpected value and causing bugs that might not be discovered immediately, Rust doesn’t compile this code at all and prevents misunderstandings early in the development process.
4. A final reason Rust doesn’t allow us to index into a `String` to get a character is that indexing operations are expected to always take constant time (O(1)). But it isn’t possible to guarantee that performance with a `String`, because Rust would have to walk through the contents from the beginning to the index to determine how many valid characters there were.

**Slicing Strings**

```rust
let hello = "안녕하세요!";
let s = &hello[0..6]; // Result : 안녕, len : 6
let s = &hello[0..4]; // Panic!
```

**Methods for Iterating Over Strings**

```rust
for c in "안녕하세요".chars() {
	// 1. 안, len : 3
	// 2. 녕, len : 3
	// 3. 하, len : 3
	// 4. 세, len : 3
	// 5. 요, len : 3
}

for c in "안녕하세요".bytes() {
	// 안 : UTF8: 236, 149, 128;
	// 녕 : UTF8: 235, 133, 144;
  // 하 : UTF8: 237, 149, 144;
  // 세 : UTF8: 236, 132, 176;
  // 요 : UTF8: 236, 154, 144;
}
```

**Strings Are Not So Simple**

To summarize, strings are complicated. Different programming languages make different choices about how to present this complexity to the programmer. Rust has chosen to make the correct handling of `String` data the default behavior for all Rust programs, which means programmers have to put more thought into handling UTF-8 data upfront. This trade-off exposes more of the complexity of strings than is apparent in other programming languages, but **it prevents you from having to handle errors involving non-ASCII characters later in your development life cycle**.

---

### [HashMap](https://doc.rust-lang.org/std/collections/struct.HashMap.html)

**Creating a New Hash Map**

All of the **keys** must have the **same type**, and all of the **values** must have the **same type**.

```rust
/*
Note that we need to first use the HashMap from the collections portion of the std lib. 
Of our three common collections, this one is the least often used, 
so it’s not included in the features brought into scope automatically in the prelude.
*/
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

```rust
let teams = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let mut scores: HashMap<_, _> =
	teams.into_iter().zip(initial_scores.into_iter()).collect();
```

**Hash Maps and Ownership**

```rust
let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);

// Compile failed
~~println!({}, field_name);~~
~~println!({}, field_value);~~
```

**Accessing Values in a Hash Map**

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
    
match score {
    Some(p) => println!("has value {}", p),
    None => println!("has no value"),
}

//-----------------------------------------------------------------------------------

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

**Updating a Hash Map**

Overwriting a Value

```rust
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores); // {"Blue": 25}
```

Only Inserting a Value If the Key Has No Value ([Entry](https://doc.rust-lang.org/std/collections/hash_map/enum.Entry.html))

```rust
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores); // {"Blue": 10, "Yellow": 50}
```

Updating a Value Based on the Old Value

```rust
let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
	let count = map.entry(word).or_insert(0);
	*count += 1;
}

println!("{:?}", map); // {"world": 2, "hello": 1, "wonderful": 1}
```

**Hashing Functions**

By default, `HashMap` uses a hashing function called SipHash that can provide resistance to Denial of Service (DoS) attacks involving hash tables[1](https://doc.rust-lang.org/book/ch08-03-hash-maps.html#siphash). This is not the fastest hashing algorithm available, but the trade-off for better security that comes with the drop in performance is worth it. If you profile your code and find that the default hash function is too slow for your purposes, you can switch to another function by specifying a different *hasher*. A hasher is a type that implements the `BuildHasher` trait. We’ll talk about traits and how to implement them in Chapter 10. You don’t necessarily have to implement your own hasher from scratch; [crates.io](https://crates.io/) has libraries shared by other Rust users that provide hashers implementing many common hashing algorithms.

**[Hasher](https://doc.rust-lang.org/std/hash/trait.Hasher.html)**

```rust
pub trait Hasher {
    fn finish(&self) -> u64;
    fn write(&mut self, bytes: &[u8]);
 
    fn write_u8(&mut self, i: u8) { ... }
    fn write_u16(&mut self, i: u16) { ... }
    ...
}
```