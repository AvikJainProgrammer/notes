https://www.youtube.com/watch?v=pnnx1bkFXng

The Eco system. 
	- This is for system programming
	- We already have c and rust
	- In contract go is a grabage collected language
	- Some project require that you do not perform grabage colleciton
	- Grabage Collection is non deterministic (what does that mean?)
	- Zig is there for people who feel rust is too hard and C is too unsafe.  
C
	- OG language
	- best language in terms of power
	- everything like router or fridge wirtten in c
	- what ever the kernal allows can be done
	- It was just an abstraction for assembly 
		- Print f and scan fa was added and everything was buffered in the heap (what does that mean what is the significance of that ?) 
Rust
	- designed with faliures of C in mind
	- C is not a memory safe language
	- It has a feature that makes you define the different kinds of output you function can return. Like a function can return an error and a normal value
The practical
	- The presenter give example of XXD binary, apprantly its a good exercise to learn a new programming language. 
		- Basically you pump in some data and a text comes out. 
		- Forces you to do basic loop, memory view, how to opne a file 
	- Main feature that separates zig from C :
		- Zig tries to prevent what is called ghost allocation , all the allocations will be the one you ask it to do. 
		- You can write your own memory allocators. 
		- You need to free the memory when you allocate and there is a defer keyword that helps you do that. 
			- defer is run when the function scope is about to do out. 
		- Zig requires you to handle all the error you get. 
			- When you have to mention the return type and he error return type of the function. 
		- Its not explicitly memory safe language
			- Allows a dangling point (what is that)
		- C lets you go out of bound of an array but zig does not
		 
https://www.youtube.com/watch?v=ZNypGSqpwdc&list=PL0-BgRHrP_sMpEvPImEbLrLKvx2Swci_6
The presenter says before teaching he is going to assume that we know things like
	- What is a stack
	- What is a heap
	- What are pointers
	- What are functions (I already know that) 

Here is the explanation from claude:

Good foundation to rebuild — Zig leans on these concepts constantly (allocators, slices, optionals), so getting them solid now will make the tutorial click much faster.

## Memory: two regions that matter

When your program runs, it gets memory organized into a few segments, but two matter most for you right now:

**The Stack**
- Fast, automatically managed, limited in size (often a few MB).
- Every time you call a function, a "stack frame" is pushed containing its local variables, parameters, and return address.
- When the function returns, that frame is popped — the memory is instantly reclaimed. No manual work needed.
- Because it's just moving a pointer up/down, it's extremely fast.

```c
void foo() {
    int x = 5;      // x lives on the stack
    int arr[10];    // this whole array lives on the stack too
}                   // when foo() returns, x and arr are gone
```

**The Heap**
- Larger, but *you* (or an allocator) manage it manually.
- Memory persists until explicitly freed — it doesn't disappear when a function returns.
- Slower than stack allocation, and mismanaging it causes real bugs.

```c
int *p = malloc(sizeof(int) * 10); // heap allocated
// ... use p ...
free(p); // you MUST do this or it's a memory leak
```

**Why the distinction exists:** if you want data to *outlive* the function that created it (e.g., return an array from a function, or store something long-term), it can't live on the stack — because that stack frame disappears the moment the function returns. It has to live on the heap.

## Pointers — the core idea

A pointer is just a variable whose value is a memory address.

```c
int x = 42;
int *p = &x;   // p holds the ADDRESS of x
*p = 100;      // dereference p, this changes x's value too
printf("%d", x); // prints 100
```

- `&x` → "give me the address of x"
- `*p` → "give me the value stored at the address p points to" (dereference)
- `int *p` → declares p as "a pointer to an int"

## The bug categories pointers create (know these — Zig fixes most of them)

**1. Dangling pointers** — pointing to memory that's already been freed or gone out of scope.
```c
int *danger() {
    int x = 10;
    return &x; // BUG: x is on the stack, dies when function returns
}
```

**2. Null pointers** — pointing to nothing (address 0), and dereferencing it crashes.
```c
int *p = NULL;
*p = 5; // crash: segfault
```

**3. Memory leaks** — heap memory you forgot to `free()`.

**4. Use-after-free** — dereferencing a pointer after you already `free()`d it.

**5. Pointer arithmetic mistakes** — `p + 1` moves by `sizeof(*p)` bytes, not 1 byte, and going out of bounds is undefined behavior with no safety net.

C gives you zero protection against any of these. This is exactly the pain Zig is designed to reduce (not eliminate — Zig is still manual memory management, but with much better tools).

## Pointers vs arrays vs "how big is this thing"

In C, a raw pointer has **no idea how many elements it points to**. `int *p` could be pointing to 1 int or 1000 — the pointer itself carries no length information. That's why C functions need you to separately pass a length (`memcpy(dst, src, n)`).

This matters a lot for Zig: Zig introduces **slices** (`[]T`) which are a pointer *plus* a length bundled together, precisely to fix this "pointer doesn't know its own size" problem. When your tutorial talks about slices, mentally connect it to: "this is a pointer that actually knows how many elements it has."

## Pointer to pointer, and passing by reference

```c
void increment(int *p) {
    *p = *p + 1;
}
int main() {
    int x = 5;
    increment(&x); // pass address so the function can modify the original
    // x is now 6
}
```

This pattern — "pass a pointer so a function can modify the caller's variable" — is the whole reason pointers exist as a *parameter-passing* tool, beyond just heap management.

## How this maps onto what you'll see in Zig

A few terms to pre-load so the tutorial doesn't trip you up:

| C concept | Zig equivalent / difference |
|---|---|
| `malloc`/`free` | Explicit **allocators** you pass around (`std.heap.page_allocator`, etc.) — no hidden global allocator |
| `NULL` pointer, forgetting to check it | **Optional types** (`?T`) — the compiler forces you to handle the "might not exist" case |
| pointer + separate length variable | **Slices** (`[]T`) — pointer and length bundled, bounds-checked in debug mode |
| undefined behavior on overflow, oob access | Zig checks many of these at runtime in debug/safe builds and traps instead of silently corrupting memory |
| stack frame, local variable lifetime | Same concept, same rules — Zig doesn't change stack semantics |
| no built-in "did I free this already" check | Zig still won't catch all of this for you, but tools like the general purpose allocator (`GeneralPurposeAllocator`) do leak/double-free detection in debug builds |

The mental model transfers almost directly — Zig didn't reinvent stack/heap/pointers, it just wrapped pointers with more information (length, optionality) and made memory management explicit rather than implicit (no hidden `new`/constructors doing allocation behind your back).

If you want, tell me which part of the Zig tutorial you're on (allocators? slices? optionals?) and I can walk through how it specifically builds on what's above.
