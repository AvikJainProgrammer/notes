# Rust Notes

Rust as explained by harkirat: Rust Totorial for Beginners - Full Course

## Terms related to Rust
- Borrow Checker
- Idomatic Rust

## Preface
In Rust, if your program complies it probably works.
You can't segfault if you don't have null.
	-nfs what is segfault

Rust doesn't hide complexity from developers it offers them the right tools to manage all the complexity.

## Syllabus for this course

### Easy
- Installing rust
- IDE setup
- Initalizing a project locally
- Variables
- Conditions, loops
- functions
- structs
- Enums
- Optional/result
- pattern matching
- package management

### Hard
This Video
- Memory management
- Mutability
- Stack vs heap
- Ownership
- Borrowing
- references

### Next Video
- Traits
- Generics
- lifetimes
- Multithreading
- Macros
- features/async await

## Why rust? Isn't Node'js enough ?

### Type Safety

```
let x = 1;

x = "harkirat"

console.log(x)
```

java script does not create about types. You should have script types.

In the following code you will see an error because rust is type safe

```rust
fn main() {
    let x = 1;
    x = "avik" # this will result in an error
    println!("Hello, world!");
}
```

Similar to rust c++ does not allow that.

### Systems Language

It is a systems language.
There are some systems resource that a system programming languge gives you access to.

For example.
RAM

Say you wanted to write a web rtc protocol, you wouldn't write it in java script you would write it in rust or c
	-nfs what is web rtc anyway

For example mediasoup is written in c and rust
pion is written in golang
there is webrtc-js but it is not as widely used.

Fun fact the media soup exposes js api and then communicates with ca and rust work so you can

```
npm install mediasoup

import {mediasoup} from "mediasoup"

mediasoup.createWebteccon()
```

If you need to build a complier or a browser, of if you want to work close to the kernal

### Generally Faster

So companies perfer
- zig
- go
- rust
- C
- Ocaml

Rust have separate compliation setps,

### Concurrency

Rust allows you to do this.
It allows you to spawn threads and these threads can independently run on cores.

node is single threaded

rust/java/go lets you spawn multiple threads.

There is something called IPC => Inter process communication. - shared multiple threads for nodejs

### Memory is memory safe

Rust has a concpet of owners, borrowing and lifetimes that mie it sextremely memory safe. 1
