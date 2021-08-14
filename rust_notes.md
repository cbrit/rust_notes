# Rust Notes

## Resources

[The Rust Programming Language — The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/book/)

[Data Types - The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/book/ch03-02-data-types.html)

[What is Ownership? - The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)

## Terms

*Binary crate* - A crate that has a `main()` function entry point.

*Crate* - A rust package/library

*Expression* - Evaluates to/returns a value

*Library crate* - A Crate that contains components that can be used in other projects. Has no `main()` function entry point.

*Shadowing* - Using and existing variables name for a new variable of either the same or a different type. Useful for situations where you would normally cast.

*Statement* - Returns no value



## Conventions

- For file names, prefer hello_world.rs over helloworld.rs

- Chain functions on a new line:

  ```rust
  lib::fn()
  	.another_fn()
  	.yet_another_fn();
  ```



## Commands

- Compile 
  - `rustc file_name.rs`. `rustc` places the executable in the current directory.
  - `cargo build` compiles to target/debug/file_name
  - `--release` flag places the executable at `target/release/file_name` and optimizes the code.
- Run
  - `cargo run`
- Validate
  - `cargo check` validate without producing an executable. Faster than compiling.
- Create a project
  - `cargo new project_name`
- Get documentation
  - `cargo doc --open`. Will build dependencies' documentation and open it in the browser.

## Ecosystem

[crates.io](https://crates.io) open source crates repository. 

Rust keeps a local list of available packages in crates.io, called the *registry*.



## Metadata and Dependencies

- `Cargo.toml`:

  ```toml
  [package]
  name = "guessing_game"
  version = "0.1.0"
  authors = ["Your Name <you@example.com>"]
  edition = "2018"
  
  # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
  
  [dependencies]
  rand = "0.8.3"
  ```

  

Dependencies in Cargo.toml use semantic versioning

- `Cargo.lock`

  Locks in dependency versions to make builds consistent across environments. `cargo update` will ignore the lock file and update crates.

## Language

### Variables

Defined with `let`.

Variables are *immutable by default*.

To make one mutable, use `let mut`.

### Functions

Definition

```rust
fn my_fn(my_param: u32) -> u32 {
	// -> u32 indications the return time is an unsigned 32 bit integer.
}
```



`TypeName::function()` to run a function from a type. Basically method dot syntax.

Pass argument by reference uses `&` prefix.

Immutable argument can be made mutable in the context of the function it is passed to by prefixing `mut`. This line passes a variable guess by reference and makes it mutable:

```rust
io::stdin()
	.read_line(&mut input_str);
```

### Syntax



### Enumerations (enums)

Values are called *variants*

Common enum `Result` is used for indicating success/failure (`Ok`) and capturing errors (`Err`) for handling

### Expressions

`loop {}`

- Essentially `while (true) {}`

- Can return a value with `break` and therefore be assigned to a variable:

  ```
  let mut counter = 0;
  
  let ten = loop {
  	counter += 1;
      
      if counter == 10 {
      	break counter;
      }
  }
  ```

  

`match {}`

- "A `match` expression is made up of *arms*. An arm consists of a *pattern* and the code that should be run if the value given to the beginning of the `match` expression fits that arm’s pattern."

- Like a `switch` statement but more powerful

- Used for handling errors after an expression 

  ```rust
  let guess: u32 = match guess.trim().parse() {
  	Ok(num) => num,
  	Err(_) => continue,
  };
  ```

Number **ranges** 

- Expressed as `x..y`, where x is the inclusive lower bound, and y is the exclusive upper bound.

Variables

- Variables are expressions in Rust. They can be used without a `return` keyword if they are the last expression in a function. This is a valid function (notice it also doesn't need a semi-colon):

  ```
  fn do_stuff() -> u32 {
  	5
  }
  ```

  

### Macros

Suffixed with `!`

### Types

[Data Types - The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/book/ch03-02-data-types.html)

## Key Rust Concepts

### Ownership

[What is Ownership? - The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)

### Shadowing

Allows a variable to be created with the same name as another. The type may be changed.

```
// This is valid. It uses shadowing.
let greeting = "hello";
let greeting: u32 = greeting.len(); // : u32 can be removed and Rust will infer the type.

// mut does not allow the same behavior
let mut greeting = "hello";
greeting = greeting.len(); // Throws an error
```

