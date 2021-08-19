## Contributing

If you are interested in contributing to this post, it's in a public repository in [my GitHub](https://github.com/cbrit/rust_notes/tree/main). If you do decide to contribute, keep in mind this is supposed to be a concise summary and reference, and it is not a rewrite of the Rust Book, even though it contains many references to quotations or code blocks from it. Thanks!

## Resources

[The Rust Programming Language — The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/book/)

[Data Types - The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/book/ch03-02-data-types.html)

[What is Ownership? - The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)

## Terms

*Binary crate* - A crate that has a `main()` function entry point.

*Borrow* - When a function is passed a value by reference. Ownership is not taken, and the value cannot be changed unless the parameter is `&mut`.

*Closure* - An anonymous function that can be saved in a variable. Similar to a C# lambda.

*Crate* - A tree of modules that produces a library or executable.

*Expression* - Evaluates to/returns a value

*Iterator Adaptor* - A method defined the `Iterator` trait for changing one `Iterator` kind to another.

*Library crate* - A Crate that contains components that can be used in other projects. Has no `main()` function entry point.

*Lifetime* - The scope for which a reference is valid.

*Memoizatoin* - Also called *lazy evaluation*. The pattern of storing a closure in a struct, only evaluating the result at the time it is needed, and then caching the result for future use.

*Method* - A function defined inside of a struct, enum, or trait. First parameter is always `self`. Methods can borrow or take ownership like normal functions.

*Module*s - Let you control the organization, scope, and privacy of paths. Included using `use`

*Monomorphization* - The process of turning generic code into specific code by filling in the concrete types that are used when compiled. The Rust compiler does this to prevent overhead from generics.

*Shadowing* - Using and existing variables name for a new variable of either the same or a different type. Useful for situations where you would normally cast.

*Statement* - Returns no value

*Trait* - Similar to an interface. Abstracts the types that implement the trait.

*Zero-cost abstractions* - An abstraction Rust provides that introduces no additional overhead. For example, an `Iterator` over using a `for` loop.

## Conventions

- For file names, prefer hello_world.rs over helloworld.rs

- Chain functions on a new line:

  ```rust
  lib::fn()
  	.another_fn()
  	.yet_another_fn();
  ```

- For function signatures that take strings or other array-like values, prefer to use slices. 

- Prefer slicing a string over indexing into it. See [Slicing Strings](https://doc.rust-lang.org/book/ch08-02-strings.html#slicing-strings) in the Rust Book for an explanation.

- If an external function is needed, include its parent and call it with `parent::function();` to disambiguate it from local functions.

- "Using primitive values when a complex type would be more appropriate is an anti-pattern known as *primitive obsession*." - Rust Book

- Place function tests in the same file as where they are defined, in the `tests`  module.

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



### Closures

Closures are anonymous functions that can be saved in a variable.

Syntax:

```rust
let x = 5;

let add_one = |num| {
	num + 1
};

// Other options
// let add_one = |num| num + 1;

// Almost never necessary annotate types because the compiler can infer them and
// closures are not exposed to a public API. However, you can do it:
// let add_one = |num: i32| -> i32 {...} 

add_one(x); // returns 6
```

- The Rust compiler will infer the parameter types from the first usage. If subsequent calls to the closure use different parameter types, the compiler will throw an error.

- Rust considered each closure to have it's own unique anonymous type, even if the signatures are the same. 

- In order to  have a closure in a struct, you need a type annotation. This is because structs must know what types their fields are.

- For closure fields, we use generics and trait bounds to define the closure type.

- `Fn` traits are defined in the standard library. They are `Fn`, `FnOnce`, and `FnMut`

- Example of a struct with a closure field:

  ```rust
  // "where T is a closure that takes an u32 and returns an u32"
  struct Cacher<T>
  where
      T: Fn(u32) -> u32,
  {
      calculation: T,
      value: Option<u32>,
  }
  ```

  The `value` field behavior:

  ```rust
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
  ```

  This pattern accomplishes *memoization* or *lazy evaluation* (see the terms section). When a developer needs to the resulting value from the closure, they will use the `value` fields, which executes the closure if it hasn't been done before, otherwise, returns the cached result.

  Beware, this struct will store the cached value of the first result with whatever argument it is given. Future attempts to pass a different argument will result in the same value as the first.

  We could get around this by making `value` a hash map, where the arguments are keys and the resulting values are the hash map values.

- Examples like the previous one can be improved by using Generic parameters and return types.

- Unlike functions, closures can capture their environment and access variable in the scope in which they are defined.

  This behavior means closures are not suitable for every scenario, because capturing the environment comes with memory overhead.

  >Closures can capture values from their environment in three ways, which directly map to the three ways a function can take a parameter: taking ownership, borrowing mutably, and borrowing immutably. These are encoded in the three `Fn` traits as follows:
  >
  >- `FnOnce` consumes the variables it captures from its enclosing scope, known as the closure’s *environment*. To consume the captured variables, the closure must take ownership of these variables and move them into the closure when it is defined. The `Once` part of the name represents the fact that the closure can’t take ownership of the same variables more than once, so it can be called only once.
  >- `FnMut` can change the environment because it mutably borrows values.
  >- `Fn` borrows values from the environment immutably.
  >
  >Rust Book Chapter 13

- The Rust compiler will infer which `Fn` traits are implemented based on the usage of the captured variables.

### Collections

Vectors `Vec<T>`

- A list of values of the **same type** stored next to each other in memory.

- Instantiated with `let v: Vec<i32> = Vec::new();`

- Rust can also infer the type during definition with the vec macro: `let v = vec![1, 2, 3];`

- Append values to vectors with the push method: `v.push(5);`. The vector must be mutable to do this.

- For reading, indexing can cause a panic if the index is out of range, or you can use the `get` method which returns an `Option`

  ```rust
  let v = vec![1, 2, 3, 4, 5];
  
  let does_not_exist = &v[100];    // panic!
  let does_not_exist = v.get(100); // No panic. Returns None.
  ```

- A nice trick is to make a vector of an enum type when you need values of different types in a vector:

  ```
  enum MyEnum {
  	Int(i32),
  	Bool(bool),
  	String(String),
  }
  
  // Vectors elements must all be the same type
  let row = vec![
  	MyEnum::Int(10),
  	MyEnum::Int(5),
  	MyEnum::Bool(true),
  	MyEnum::String(String::from("Hi")),
  	MuEnum::String(String::from("Bye")),
  ];
  ```

Strings

- Referring to the `String` type (as apposed to the `&str` slice)

- A string is a collection of bytes, therefore it falls under this category

  In fact, is a wrapper of a `Vec<u8>`, and as such can be indexed like a normal vector.

- Remember that a string literal is a slice, not a `String`. These are equivalent:

  ```rust
  // The to_string() method is available to any type that implements
  // the Display trait.
  let s1 = "This is a literal".to_string();
  let s2 = String::from("This is a literal"); 
  ```

- Like a vector, a `String` can be appended to.

- The `+` operator can concatenate strings. Under the hood it's using the `add()` method, which takes ownership of the argument on the left.

  ```rust
  let s1 = String::from("Hello");
  let s2 = String::from(", World!");
  let s3 = s1 + &s2; // Notice the second argument is a reference. This line moves s1.
  ```

  The `add` method signature is `fn add(self, s: &str) -> String`

- The `format!` macro can be used to format a string:

  ```rust
  let s1 = String::from("Hello");
  let s2 = String::from(", World!");
  let s3 = format!("{}{}", s1, s2);
  ```

  

Hash Maps

- A familiar data structure known by other names such as map or dictionary.

  ```rust
  // Include
  use std::collections::HashMap;
  
  // Initializing
  let mut hm = HashMap::new();
  
  // Add pairs
  hm.insert(String::from("Username"), String::from("atrooo"));
  ```

- All keys must be of the same types, as well as all values. In the previous code block, Rust infers that the type is `HashMap<String, String>`. It could have just as easily been `HashMap<i32, bool>`.

  It can be explicitly defined when initializing, just like any other type:

  ```rust
  let mut hm: HashMap<String, i32> = HashMap::new();
  ```

- Hash maps take ownership of values inserted into them.

- Accessing:

  ```rust
  // Get method
  let key = String::from("Username");
  let int: i32 = hm.get(&key);
  
  // Or iterate with tuple
  for (key, value) in &hm {
  	println!("{}: {}", key, value); // Username: atrooo
  }
  ```

  

### Enumerations (enums)

Values are called *variants*

Common enum `Result` is used for indicating success/failure (`Ok`) and capturing errors (`Err`) for handling

Defined as follows:

```rust
enum MyEnum {
	Variant1,
	Varient2,
};
```

Can be combined with structs to store data with meaingful descriptions in the type name

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
```

Rust provides a short hand for the previous patter, allowing define data excepted directly in the variant definition:

```rust
enum IpAddr {
    V4(String),
    V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));
```

The previous two blocks of code result in the same data structure in the `home` variable.

Additionally, each variant can accept different types and amount of data

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```

The advantage of using an `enum` is a better UX when writing data structures and easier to read code. If you had two write two structs `V4` and `V6`, you would then need separate function signatures for each type. 

Just like structs, enums can have methods and associated functions.

##### Special enum: Option<T>

In place of the concept of null and not-null values, Rust uses the `Option<T>` enum to represent a value that can be either present or not present. It's defined in the standard library as 

```rust
enum Option<T> {
    Some(T),
    None,
}
```

You can use its variant Some and None without the `Option::` prefix. 

The type passed to the generic type parameter `<T>` means that `Some` can hold any type.

By doing this, rather than allowing each variable to have a `null` state, the compiler can treat variables that have the potential to not be present as different than the regular type. Additionally, the compiler will force the user to handle every variant in the Options before compiling. This prevents attempts to execute code on a missing value.

### Error Handling

- By default, Rust panics when it encounters an error it can't recover from, and *unwinds* back up the backtrace, cleaning up the variables created.

- There is a `panic!` macro for intentionally panicing

- When the `RUST_BACKTRACE` environment variable is set to `1` Rust will output a backtrace to the standard output.

- The `Result` enum can be used to handle errors via it's `Err(E)` variant.

  ```rust
  let f = File::open("fakefile");
  
  // fakefile does not exist
  let f = match f {
      Ok(file) => file,
      Err(error) => panic!("Something went wrong: {:?}", error),
  };
  ```

- `Result` has two helper methods, `unwrap` and `expect` to help avoid the boiler plate `match` expression that handles the `Result` variants. `unwrap` will return the value if the `Ok` arm is taken, or panic with the default error message. `expect` does the same thing but allows you to pass your own error message. This code does the same thing as the previous example:

  ```rust
  let f = File::open("fakefile").expect("Something went wrong.");
  
  // let f = File::open("fakefile").unwrap();
  ```

- Errors can be propagated by returning them

  ```rust
  let f = File::open("fakefile");
  
  let f = match f {
      Ok(file) => file,
      Err(error) => return error,
  };
  ```

  This leaves it up to the calling function to handle the error.

- The `?` operator is shorthand for accomplishing this same functionality:

  ```rust
  let f = File::open("fakefile")?
  ```

- [Guidelines on when to panic](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html)

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

- Matches are exhaustive. If there is a variant unhandled, the compiler will throw an error.

- Use the `_` placeholder variant to handle unlisted variants. Sort of like a `default` block in a switch statement. Matches any value.

- Patterns are matched in the order they are written. Placing `_` as the first arm would match all inputs.

`if let` 

- Syntactic sugar for unexhaustive matching

- These are equivalent:

  ```rust
  let some_u8_value = Some(0u8);
  
  match some_u8_value {
      Some(3) => println!("three"),
      _ => (),
  }
  ```

  ```rust
  let some_u8_value = Some(0u8);
  
  if let Some(u8) = some_u8_value {
      println!("three");
  }
  
  // If some_u8_value were None, nothing would happen.
  ```

- You can follow and `if let` with an `else` block.

Number **ranges** 

- Expressed as `x..y`, where x is the inclusive lower bound, and y is the exclusive upper bound.

Variables

- Variables are expressions in Rust. They can be used without a `return` keyword if they are the last expression in a function. This is a valid function (notice it also doesn't need a semi-colon):

  ```rust
  fn do_stuff() -> u32 {
  	5
  }
  ```



### Iterators

- Similar to iterators in any other language that has them. They return sequences of data one at a time.

- Iterators must implement the `Iterator` trait by implementing the following items (from standard library definition):

  ```rust
  pub trait Iterator {
      type Item;
  
      fn next(&mut self) -> Option<Self::Item>;
  
      // methods with default implementations elided
      // ...
  }
  ```

  For example

  ```rust
  struct Counter {
      count: u32,
  }
  
  impl Counter {
      fn new() -> Counter {
          Counter { count: 0 }
      }
  }
  
  impl Iterator for Counter {
      type Item = u32;
  
      fn next(&mut self) -> Option<Self::Item> {
          if self.count < 5 {
              self.count += 1;
              Some(self.count)
          } else {
              None
          }
      }
  }
  ```

- *Iterator adaptors* change one iterator into another. A common one used in Rust is `filter`
- The `filter` method on an iterator takes a closure that performs a check on each item from the iterator and returns a `bool`. If the resulting value is `true`, the item is included in the resulting iterator from the `filter` method. If the closure result is `false`, the item is not included in the resulting iterator.
- 

### Generics

- Types or Functions defined with generic parameters:

  ```rust
  struct Point<T> {
      x: T,
      y: T,
  }
  
  // T must have the PartialOrd trait for the
  // > operation to work on it. Since not every
  // type can be orders, and thus compared, this
  // function will throw an error.
  fn largest(v: Vec<T>) -> T {
      let largest = v[0];
      
      for value in v {
          if v > largest {
              largest = v;
          }
      }
  }
  ```

- Rust will infer the type from the value provided. If you were to pass `x = 5` and `y = 4.0` to the struct in the previous example, it would throw an error because Rust inferred that T should be `i32` from the first value.



### Keywords

`move` - Force ownership of a captured variable (move from reference to value). Used with closures. 

`pub` - Makes item publicly available to other crates.



### Macros

Suffixed with `!`

`panic!` - Panics with the provided error message.

`println!` - Writes to stdout

`println!` - Writes to stderr

`vec!` - shorthand for a `Vec<T>` constructor. 

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



### Tests

- Run with `cargo test`

- Place tests in a `tests` module

- Mark the module with the `#[cfg(test)]` annotation, and each test function within it with `#[test]`

  ```rust
  #[cfg(test)]
  mod tests {
      use super::*;
      
          #[test]
      fn one_result() {
          let query = "duct";
          let contents = "\
  Rust:
  safe, fast, productive.
  Pick three.";
  
          assert_eq!(vec!["safe, fast, productive."], search(query, contents));
      }
  }
  ```

  `super` refers to the module directly above `self` in the path. The line `use super::*` is importing all sibling crates.



### Traits

- Rusts notion of an interface.

- Defining:

  ```rust
  pub trait Summary {
      fn summarize(&self) -> String;
  }
  ```

- Cannot implement external traits on external types. Either the trait or the type must be local to your crate.

- Unlike a regular interface, traits can have default behavior defined on them:

  ```rust
  pub trait Summary {
      fn summarize(&self) -> String {
          String::from("(Read more...)")
      }
  }
  ```

- The default implementation can be overwritten. In such a case, the default implementation cannot be used.

- Traits be used as parameters:

  ```rust
  pub fn notify(item: &impl Summary) {
      println!("Breaking news! {}", item.summarize());
  }
  
  // Long form ("trait bound")
  pub fn notify<T: Summary>(item: &T) {
      println!("Breaking news! {}", item.summarize());
  }
  ```

  This function can take any type that implements the Summary trait as a parameter.

  The "trait bound" syntax allows you to use pass parameters of different types on function that take more than one parameter, so long as they both implement the trait.

- Parameter that allows multiple kinds of traits with the `+` operation:

  ```rust
  pub fn notify(item: &(impl Summary + Display)) {
  
  // trait bound
  pub fn notify<T: Summary + Display>(item: &T) {
  ```

- Since this syntax can get really verbose, there is a `where` clause that makes things neater:

  ```rust
  fn some_function<T, U>(t: &T, u: &U) -> impl Clone
  	where T: Display + Clone,
  		  U: Clone + Debug
  {
  ```

- Trait can be a return type. Notice the use of the `Clone` trait as the return type in the previous code block.

### Types

[Data Types - The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/book/ch03-02-data-types.html)

- Because primitive types have a known size, rust will store them on the stack. Passing them between scopes creates a copy.
- Types of an unknown size (like String) will be stored on the heap. Passing them between scopes is either accomplished by moving ownership or borrowing with a reference.

## Key Rust Concepts

### Lifetimes

[Validating References with Lifetimes - The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html)

Every reference in Rust has a lifetime, "which is the scope for which that reference is valid".

Meant to prevent dangling references.

Rust has a *borrow check* meant to check that all borrows are valid and don't attempt to use any values that are out of scope. 

**Lifetime annotations** allow you to constrain values in function to certain lifetimes. The reason this is needed is that if a function takes two borrow parameters, and then returns a borrowed value, it doesn't know whether the value it's returning is still in scope. If you constrain both parameters and the return type to have the same lifetime, the compiler can then tell whether the function is valid.

Syntax for lifetime annotations:

```rust
// lifetime 'a
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```



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

