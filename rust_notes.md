# Rust Notes

## Resources

[The Rust Programming Language — The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/book/)

[Data Types - The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/book/ch03-02-data-types.html)

[What is Ownership? - The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)

## Terms

*Binary crate* - A crate that has a `main()` function entry point.

*Borrow* - When a function is passed a value by reference. Ownership is not taken, and the value cannot be changed unless the parameter is `&mut`.

*Crate* - A rust package/library

*Expression* - Evaluates to/returns a value

*Library crate* - A Crate that contains components that can be used in other projects. Has no `main()` function entry point.

*Method* - A function defined inside of a struct, enum, or trait. First parameter is always `self`. Methods can borrow or take ownership like normal functions.

*Shadowing* - Using and existing variables name for a new variable of either the same or a different type. Useful for situations where you would normally cast.

*Statement* - Returns no value

*Trait* - Similar to an interface. Abstracts the types that implement the trait.



## Conventions

- For file names, prefer hello_world.rs over helloworld.rs

- Chain functions on a new line:

  ```rust
  lib::fn()
  	.another_fn()
  	.yet_another_fn();
  ```

- For function signatures that take strings or other array-like values, prefer to use slices. 

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

Immutable argument can be made mutable in the context of the function it is passed to by prefixing `mut`. This line passes a variable `input_str` by reference and makes it mutable:

```rust
io::stdin()
	.read_line(&mut input_str);
```



### Enumerations (enums)

Values are called *variants*

Common enum `Result` is used for indicating success/failure (`Ok`) and capturing errors (`Err`) for handling

### Expressions

`loop {}`

- Essentially `while (true) {}`

- Can return a value with `break` and therefore be assigned to a variable:

  ```rust
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

  ```rust
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

#### Ownership Rules

- Each value in Rust has a variable that’s called its *owner*.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

#### Passing by Value

Values instantiated on the heap get dropped when their owning variable goes out of scope.

If the variable is assigned to a second variable, the second variable assumes ownership of the pointer to the heap (rather than making a copy), and the first variable is no longer valid. This is called *moving*.

For deep copying, the clone() method exists. 

Moves *do not* occur for values stored on the stack (scalar values, all values with a known size at compile time), because making copies is cheap.

So, *moving* occurs with heap values, *copying* occurs with stack values. Copying is functionally the same thing as using clone().

Ownership rules work the same with assigning to a variable and passing  to a function. The value is either copied or moved. If moved, the owner is now scoped within the function, and will be dropped when the function ends unless returned to the parent scope.

Returning a Drop (trait of heap values) moves ownership to the variable that captures the return value.

#### Passing by Reference

Allows *borrowing* of a value by passing the pointer to a function.

Syntax for borrowing is 

```rust
fn main()  {
	let s = String::("hello");

	my_function(&s);
}

fn my_function(s: &String) {
	// do something
}
```



A borrowed value cannot be changed unless it is explicitly mutable in both the owner's definition, and the function parameter:

```rust
fn main()  {
	let mut s = String::("hello");

	my_function(&s);
}

fn my_function(s: &mut String) {
	// do something
}
```

- Only one mutable reference to a value can exist in a scope

- You cannot have a mutable reference to a value that another variable has an immutable reference to. 

  > Users of an immutable reference don’t expect the values to suddenly change out from under them!
  >
  > *Rust Book Chapter 4*

- Multiple immutable references are ok

- This one is weird. The scope of a reference ends after the last time the reference is used. So while this is not okay:

  ```rust
      let mut s = String::from("hello");
  
      let r1 = &s; // no problem
      let r2 = &s; // no problem
      let r3 = &mut s; // BIG PROBLEM
  
      println!("{}, {}, and {}", r1, r2, r3); // Immutable references are still in scope
  ```

   This is ok:

  ```rust
      let mut s = String::from("hello");
  
      let r1 = &s; // no problem
      let r2 = &s; // no problem
      println!("{} and {}", r1, r2);
      // r1 and r2 are no longer used after this point and so their scope ends
  
      let r3 = &mut s; // no problem
      println!("{}", r3);
  ```

Recap,

- At any given time, you can have *either* one mutable reference *or* any number of immutable references.
- References must always be valid.

### Shadowing

Allows a variable to be created with the same name as another. The type may be changed.

```rust
// This is valid. It uses shadowing.
let greeting = "hello";
let greeting: u32 = greeting.len(); // : u32 can be removed and Rust will infer the type.

// mut does not allow the same behavior
let mut greeting = "hello";
greeting = greeting.len(); // Throws an error
```

### Slices

Work similarly to slices in Golang. 

References part of a string

`&my_str[5..12] // slice from index 5 through 11 `

`&my_str[..12] // slice from beginning of string through index 11`

`&my_str[5..] // slice from index 5 through end of string`

`&my_str[..] // slice of entire string`

You can also use variables for the bounds:

```rust
let my_str = String::from("Hello, slices!");
let len = my_str.len();

let slice = &my_str[7..len]; // slice containing "slices!"
```

String literals are slices. String literal type keyword is `str`, so a function can receive/return a slice by using the syntax `&str`.

Slice definitions for other types look like `&[u32]`, `&[bool]`, etc.

### Structs

Like structs in other languages.

Each struct defined is its own type.

Defining:

```rust
struct MyStruct {
	name: String,
	email: String,
	age: u32,
};
```

Instantiating:

```rust
let mut instance = MyStruct {
	name: String::from("Collin"),
	email: String::fromt("someemail@gmail.com"),
	age: 100,
};
```

Accessing:

```rust
println!("Hello, {}!", instance.name);

instance.age = instance.age + 1; // This will throw a compiler error if instance is not mutable
```

Entire instance must be mutable or immutable. You cannot mark only certain fields as mutable.

Shorthand for auto-filling fields with matching function parameters:

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email, 		// Field and parameter have same name
        username,	// Field and parameter have same name
        active: true,
        sign_in_count: 1,
    }
}
```

"Update syntax", copy fields from another struct 

```rust
let user2 = User {
	email: String::from("another@example.com"),
	username: String::from("anotherusername567"),
	..user1	// Remaining fields will have the same values as those in user1
};
```

#### Structs: Methods

Methods can be defined on structs with the `impl` ("implements") block. Methods take `self` as their first argument. Borrowing and ownership work as normal. Methods are called with dot syntax:

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

// Given a Rectangle rect, area method can be called with rect.area();
```

#### Structs: Associated Functions

Associated functions are functions in a struct that do not take the `self` parameter, and therefore do not have an instance of the caller scoped. They are called with the double colon `::` syntax:

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
}

// Called with Rectangle::square(5)
```

Associated functions are often used as constructors that return a new instance of a the struct. The previous example creates a new Rectangle with equal width and height.