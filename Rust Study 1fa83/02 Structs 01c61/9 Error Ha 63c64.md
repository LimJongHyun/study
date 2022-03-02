# 9. Error Handling

Rust groups errors into two major categories: ***recoverable*** and ***unrecoverable*** errors. For a recoverable error, such as a file not found error, it’s reasonable to report the problem to the user and retry the operation. Unrecoverable errors are always symptoms of bugs, like trying to access a location beyond the end of an array.

Most languages don’t distinguish between these two kinds of errors and handle both in the same way, using mechanisms such as exceptions. **Rust doesn’t have exceptions.** Instead, it has the type `Result<T, E>` for recoverable errors and the `panic!` macro that stops execution when the program encounters an unrecoverable error. This chapter covers calling `panic!` first and then talks about returning `Result<T, E>` values. Additionally, we’ll explore considerations when deciding whether to try to recover from an error or to stop execution.

# [9.1 Unrecoverable Errors with `panic!`](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#unrecoverable-errors-with-panic)

[Unwinding the Stack or Aborting in Response to a Panic](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#unwinding-the-stack-or-aborting-in-response-to-a-panic)

By default, when a panic occurs, the program starts ***unwinding***, which means Rust walks back up the stack and cleans up the data from each function it encounters. But this walking back and cleanup is a lot of work. The alternative is to immediately *abort*, which ends the program without cleaning up. Memory that the program was using will then need to be cleaned up by the operating system. If in your project you need to make the resulting binary as small as possible, you can switch from unwinding to aborting upon a panic by adding `panic = 'abort'` to the appropriate `[profile]` sections in your *Cargo.toml* file. For example, if you want to abort on panic in release mode, add this:

```rust
[profile.release]
panic = 'abort'
```

```rust
fn main() {
    panic!("crash and burn");
}

----
PS C:\Works\rust\hello-rust> cargo run
   Compiling hello-rust v0.1.0 (C:\Works\rust\hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target\debug\hello-rust.exe`
thread 'main' panicked at 'crash and burn', src\main.rs:2:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
error: process didn't exit successfully: `target\debug\hello-rust.exe` (exit code: 101)
```

```rust
fn main() {
    let v = vec![1, 2, 3];
    v[99];
}

----
PS C:\Works\rust\hello-rust> cargo run
   Compiling hello-rust v0.1.0 (C:\Works\rust\hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.46s
     Running `target\debug\hello-rust.exe`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src\main.rs:4:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
error: process didn't exit successfully: `target\debug\hello-rust.exe` (exit code: 101)

----
PS C:\Works\rust\hello-rust> $env:RUST_BACKTRACE = 1                          
PS C:\Works\rust\hello-rust> cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.21s
     Running `target\debug\hello-rust.exe`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src\main.rs:4:5 
stack backtrace:
   0: std::panicking::begin_panic_handler
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\panicking.rs:517  
   1: core::panicking::panic_fmt
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\core\src\panicking.rs:101 
   2: core::panicking::panic_bounds_check
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\core\src\panicking.rs:77  
   3: core::slice::index::impl$2::index<i32>
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\library\core\src\slice\index.rs:184
   4: core::slice::index::impl$0::index<i32,usize>
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\library\core\src\slice\index.rs:15 
   5: alloc::vec::impl$16::index<i32,usize,alloc::alloc::Global>
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\library\alloc\src\vec\mod.rs:2465
   6: hello_rust::main
             at .\src\main.rs:4
   7: core::ops::function::FnOnce::call_once<void (*)(),tuple$<> >
             at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\library\core\src\ops\function.rs:227
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
error: process didn't exit successfully: `target\debug\hello-rust.exe` (exit code: 101)

----
PS C:\Works\rust\hello-rust> $env:RUST_BACKTRACE = "full"
PS C:\Works\rust\hello-rust> cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.03s
     Running `target\debug\hello-rust.exe`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src\main.rs:4:5
stack backtrace:
   0:     0x7ff666ec812e - std::backtrace_rs::backtrace::dbghelp::trace
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\..\..\backtrace\src\backtrace\dbghelp.rs:98
   1:     0x7ff666ec812e - std::backtrace_rs::backtrace::trace_unsynchronized
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\..\..\backtrace\src\backtrace\mod.rs:66
   2:     0x7ff666ec812e - std::sys_common::backtrace::_print_fmt
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\sys_common\backtrace.rs:67   
   3:     0x7ff666ec812e - std::sys_common::backtrace::_print::impl$0::fmt
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\sys_common\backtrace.rs:46   
   4:     0x7ff666ed673a - core::fmt::write
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\core\src\fmt\mod.rs:1150
   5:     0x7ff666ec63d8 - std::io::Write::write_fmt<std::sys::windows::stdio::Stderr>
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\io\mod.rs:1667
   6:     0x7ff666eca3d6 - std::sys_common::backtrace::_print
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\sys_common\backtrace.rs:49   
   7:     0x7ff666eca3d6 - std::sys_common::backtrace::print
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\sys_common\backtrace.rs:36   
   8:     0x7ff666eca3d6 - std::panicking::default_hook::closure$1
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\panicking.rs:210
   9:     0x7ff666ec9ec4 - std::panicking::default_hook
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\panicking.rs:227
  10:     0x7ff666ecaa35 - std::panicking::rust_panic_with_hook
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\panicking.rs:624
  11:     0x7ff666eca61b - std::panicking::begin_panic_handler::closure$0
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\panicking.rs:521
  12:     0x7ff666ec8a77 - std::sys_common::backtrace::__rust_end_short_backtrace<std::panicking::begin_panic_handler::closure$0,never$>
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\sys_common\backtrace.rs:141  
  13:     0x7ff666eca579 - std::panicking::begin_panic_handler
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\panicking.rs:517
  14:     0x7ff666edaf90 - core::panicking::panic_fmt
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\core\src\panicking.rs:101
  15:     0x7ff666edaf57 - core::panicking::panic_bounds_check
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\core\src\panicking.rs:77
  16:     0x7ff666ec236d - core::slice::index::impl$2::index<i32>
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\library\core\src\slice\index.rs:184
  17:     0x7ff666ec22f8 - core::slice::index::impl$0::index<i32,usize>
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\library\core\src\slice\index.rs:15
  18:     0x7ff666ec26c2 - alloc::vec::impl$16::index<i32,usize,alloc::alloc::Global>
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\library\alloc\src\vec\mod.rs:2465
  19:     0x7ff666ec1ba0 - hello_rust::main
                               at C:\Works\rust\hello-rust\src\main.rs:4
  20:     0x7ff666ec15ab - core::ops::function::FnOnce::call_once<void (*)(),tuple$<> >
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\library\core\src\ops\function.rs:227
  21:     0x7ff666ec238b - std::sys_common::backtrace::__rust_begin_short_backtrace<void (*)(),tuple$<> >
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\library\std\src\sys_common\backtrace.rs:125   
  22:     0x7ff666ec2431 - std::rt::lang_start::closure$0<tuple$<> >
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\library\std\src\rt.rs:63
  23:     0x7ff666ecae86 - core::ops::function::impls::impl$2::call_once
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\library\core\src\ops\function.rs:259
  24:     0x7ff666ecae86 - std::panicking::try::do_call
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\panicking.rs:403
  25:     0x7ff666ecae86 - std::panicking::try
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\panicking.rs:367
  26:     0x7ff666ecae86 - std::panic::catch_unwind
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\panic.rs:129
  27:     0x7ff666ecae86 - std::rt::lang_start_internal::closure$2
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\rt.rs:45
  28:     0x7ff666ecae86 - std::panicking::try::do_call
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\panicking.rs:403
  29:     0x7ff666ecae86 - std::panicking::try
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\panicking.rs:367
  30:     0x7ff666ecae86 - std::panic::catch_unwind
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\panic.rs:129
  31:     0x7ff666ecae86 - std::rt::lang_start_internal
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\/library\std\src\rt.rs:45
  32:     0x7ff666ec23ff - std::rt::lang_start<tuple$<> >
                               at /rustc/59eed8a2aac0230a8b53e89d4e99d55912ba6b35\library\std\src\rt.rs:62
  33:     0x7ff666ec1bf6 - main
  34:     0x7ff666ed9d34 - invoke_main
                               at d:\a01\_work\6\s\src\vctools\crt\vcstartup\src\startup\exe_common.inl:78
  35:     0x7ff666ed9d34 - __scrt_common_main_seh
                               at d:\a01\_work\6\s\src\vctools\crt\vcstartup\src\startup\exe_common.inl:288
  36:     0x7ffe584b7034 - BaseThreadInitThunk
  37:     0x7ffe59d82651 - RtlUserThreadStart
error: process didn't exit successfully: `target\debug\hello-rust.exe` (exit code: 101)
```

That’s a lot of output! The exact output you see might be different depending on your operating system and Rust version. In order to get backtraces with this information, debug symbols must be enabled. Debug symbols are enabled by default when using `cargo build` or `cargo run` without the `-release` flag, as we have here.

# [9.2 Recoverable Errors with `Result`](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#recoverable-errors-with-result)

Most errors aren’t serious enough to require the program to stop entirely. Sometimes, when a function fails, it’s for a reason that you can easily interpret and respond to. For example, if you try to open a file and that operation fails because the file doesn’t exist, you might want to create the file instead of terminating the process.

Recall from [“Handling Potential Failure with the `Result` Type”](https://doc.rust-lang.org/book/ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-the-result-type) in Chapter 2 that the `Result` enum is defined as having two variants, `Ok` and `Err`, as follows:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let _f = match f {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}

----
PS C:\Works\rust\hello-rust> cargo run
   Compiling hello-rust v0.1.0 (C:\Works\rust\hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.44s
     Running `target\debug\hello-rust.exe`
thread 'main' panicked at 'Problem opening the file: Os { code: 2, kind: NotFound, message: "지정된 파일을 찾을 수 없습니다." }', src\main.rs:8:23
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
error: process didn't exit successfully: `target\debug\hello-rust.exe` (exit code: 101)
```

[Matching on Different Errors](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#matching-on-different-errors)

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let _f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error)
            }
        },
    };
}

----
PS C:\Works\rust\hello-rust> cargo run
   Compiling hello-rust v0.1.0 (C:\Works\rust\hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.45s
     Running `target\debug\hello-rust.exe`
PS C:\Works\rust\hello-rust>
```

The type of the value that `File::open` returns inside the `Err` variant is `io::Error`, which is a struct provided by the standard library. This struct has a method `kind` that we can call to get an `io::ErrorKind` value. The enum `io::ErrorKind` is provided by the standard library and has variants representing the different kinds of errors that might result from an `io` operation. The variant we want to use is `ErrorKind::NotFound`, which indicates the file we’re trying to open doesn’t exist yet. So we match on `f`, but we also have an inner match on `error.kind()`.

That’s a lot of `match`! The `match` expression is very useful but also very much a primitive. In Chapter 13, you’ll learn about closures; the `Result<T, E>` type has many methods that accept a closure and are implemented using `match` expressions. Using those methods will make your code more concise. A more seasoned Rustacean might write this code instead of Listing 9-5:

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let _f = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

[Shortcuts for Panic on Error: `unwrap` and `expect`](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#shortcuts-for-panic-on-error-unwrap-and-expect)

Using `match` works well enough, but it can be a bit verbose and doesn’t always communicate intent well. The `Result<T, E>` type has many helper methods defined on it to do various tasks. One of those methods, called `unwrap`, is a shortcut method that is implemented just like the `match` expression we wrote in Listing 9-4. If the `Result` value is the `Ok` variant, `unwrap` will return the value inside the `Ok`. If the `Result` is the `Err` variant, `unwrap` will call the `panic!` macro for us. Here is an example of `unwrap` in action:

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}

----
PS C:\Works\rust\hello-rust> cargo run
   Compiling hello-rust v0.1.0 (C:\Works\rust\hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.29s
     Running `target\debug\hello-rust.exe`
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "지정된 파일을 찾
을 수 없습니다." }', src\main.rs:4:38
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
error: process didn't exit successfully: `target\debug\hello-rust.exe` (exit code: 101)
```

```rust
use std::fs::File;

fn main() {
    let _f = File::open("hello.txt").expect("Failed to open hello.txt");
}

----
PS C:\Works\rust\hello-rust> cargo run
   Compiling hello-rust v0.1.0 (C:\Works\rust\hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
     Running `target\debug\hello-rust.exe`
thread 'main' panicked at 'Failed to open hello.txt: Os { code: 2, kind: NotFound, message: "지정된 파일을 찾을 수 없습니다." }', src\main.rs:4:38
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
error: process didn't exit successfully: `target\debug\hello-rust.exe` (exit code: 101)
```

[Propagating Errors](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#propagating-errors)

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

[A Shortcut for Propagating Errors: the `?` Operator](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator)

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

The `?` placed after a `Result` value is defined to work in almost the same way as the `match` expressions we defined to handle the `Result` values in Listing 9-6. If the value of the `Result` is an `Ok`, the value inside the `Ok` will get returned from this expression, and the program will continue. If the value is an `Err`, the `Err` will be returned from the whole function as if we had used the `return` keyword so the error value gets propagated to the calling code.

There is a difference between what the `match` expression from Listing 9-6 does and what the `?` operator does: error values that have the `?` operator called on them go through the `from` function, defined in the `From` trait in the standard library, which is used to convert errors from one type into another. When the `?` operator calls the `from` function, the error type received is converted into the error type defined in the return type of the current function. This is useful when a function returns one error type to represent all the ways a function might fail, even if parts might fail for many different reasons. As long as each error type implements the `from` function to define how to convert itself to the returned error type, the `?` operator takes care of the conversion automatically.

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```

```rust
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

Reading a file into a string is a fairly common operation, so Rust provides the convenient `fs::read_to_string` function that opens the file, creates a new `String`, reads the contents of the file, puts the contents into that `String`, and returns it. Of course, using `fs::read_to_string` doesn’t give us the opportunity to explain all the error handling, so we did it the longer way first.

[The `?` Operator Can Be Used in Functions That Return `Result`](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#the--operator-can-be-used-in-functions-that-return-result)

The `?` operator can be used in functions that have a return type of `Result`, because it is defined to work in the same way as the `match` expression we defined in Listing 9-6. The part of the `match` that requires a return type of `Result` is `return Err(e)`, so the return type of the function has to be a `Result` to be compatible with this `return`.

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt")?;
}

----
PS C:\Works\rust\hello-rust> cargo run
   Compiling hello-rust v0.1.0 (C:\Works\rust\hello-rust)
error[E0277]: the `?` operator can only be used in a function that returns `Result` or `Option` (or another type that implements `FromResidual`)
   --> src\main.rs:4:36
    |
3   | / fn main() {
4   | |     let f = File::open("hello.txt")?;
    | |                                    ^ cannot use the `?` operator in a function that returns `()`
5   | | }
    | |_- this function should return `Result` or `Option` to accept `?`
    |
    = help: the trait `FromResidual<Result<Infallible, std::io::Error>>` is not implemented for `()`
note: required by `from_residual`
   --> C:\Users\cucryma\.rustup\toolchains\stable-x86_64-pc-windows-msvc\lib/rustlib/src/rust\library\core\src\ops\try_trait.rs:339:5
    |
339 |     fn from_residual(residual: R) -> Self;
    |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For more information about this error, try `rustc --explain E0277`.
error: could not compile `hello-rust` due to previous error
```

The `main` function is special, and there are restrictions on what its return type must be. One valid return type for main is `()`, and conveniently, another valid return type is `Result<T, E>`, as shown here:

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let f = File::open("hello.txt")?;

    Ok(())
}

----
warning: `hello-rust` (bin "hello-rust") generated 2 warnings
    Finished dev [unoptimized + debuginfo] target(s) in 0.09s
     Running `target\debug\hello-rust.exe`
Error: Os { code: 2, kind: NotFound, message: "지정된 파일을 찾을 수 없습니다." }    
error: process didn't exit successfully: `target\debug\hello-rust.exe` (exit code: 1)
```

The `Box<dyn Error>` type is called a trait object, which we’ll talk about in the [“Using Trait Objects that Allow for Values of Different Types”](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types) section in Chapter 17. For now, you can read `Box<dyn Error>` to mean “any kind of error.” Using `?` in a `main` function with this return type is allowed.
Now that we’ve discussed the details of calling `panic!` or returning `Result`, let’s return to the topic of how to decide which is appropriate to use in which cases.

# [9.3 To `panic!` or Not to `panic!`](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic)

So how do you decide when you should call `panic!` and when you should return `Result`?

[Examples, Prototype Code, and Tests](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#examples-prototype-code-and-tests)

panic

[Cases in Which You Have More Information Than the Compiler](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler)

```rust
use std::net::IpAddr;

fn main() {
    let _home: IpAddr = "127.0.0.1".parse().unwrap();
}
```

[Guidelines for Error Handling](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling)

~~~

[Creating Custom Types for Validation](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation)

```rust
loop {
    // --snip--

    let guess: i32 = match guess.trim().parse() {
        Ok(num) => num,
        Err(_) => continue,
    };

    if guess < 1 || guess > 100 {
        println!("The secret number will be between 1 and 100.");
        continue;
    }

    match guess.cmp(&secret_number) {
        // --snip--
}
```

**A** `Guess` **type that will only continue with values between 1 and 100**

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
```