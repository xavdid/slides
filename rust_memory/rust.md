theme: Next, 1
autoscale: true

# Rust's Approach to Memory

### Exploring Rust's most interesting feature

#### A Sandboxes learning-time production

---

# Rust has been the "hot new thing"

Why?

- extremely fast at runtime
- superb developer experience (tooling, etc)
- memory safety without garbage collection

---

# Rust has been the "hot new thing"

Why?

- extremely fast at runtime
- superb developer experience (tooling, etc)
- **memory safety** without garbage collection

Let's dig into how programs use memory and what makes Rust unique.

---

# Computers Have Memory

![](mezzanine_643.jpg)

We'll be talking about memory allocation, so it's good to understand what it is and how it flows:

1. Computers are physical boxes with a finite amount of Random Access **Memory** (`RAM`). The OS divvies available memory out to all the running programs
2. Programming **languages** are general-purpose programs that can interpret or compile bits of text into runnable programs
3. Within a **program**, most actions will store data temporarily, requiring more memory (which the language will request from the OS)

---

# Programs Use Memory

When a program needs to store data (like when it allocates a variable), the computer needs to cordon off memory. Take this JS code:

```js
let s = "I take up memory!";
```

Now the computer has reserved a couple of bytes of RAM to store that string. Most programs allocate memory constantly the place because we as programmers are not trained to use as few variables as possible (nor should we be; computers are fast).

---

# Memory Comes and Goes

If you reassign a variable, the old data is no longer accessible to the program:

```js
let s = "This string will be lost... like tears in rain";
s = "some new string!";
```

If the JS runtime didn't free the memory used by the old string, (more on that soon) then that memory would _never_ be returned to the computer. Run this program enough times and your computer would eventually run out of memory entirely! 😱

---

# Take Out the Trash

Rather than let programs grow their memory forever, most languages perform **garbage collection**.

When a data is no longer in use (like the first string before), the language marks it for deletion.[^1] Then periodically, the program will stop what it's doing and "free" all of the no-longer-used memory, returning it to the computer.

[^1]: There are a number of approaches for this, such as "reference counting". Here's a good [deep-dive on the subject](https://devguide.python.org/internals/garbage-collector/index.html)

---

# Is Garbage Collection Good or Bad?

It depends!

Because the program has to take time to track data usage and stop everything to take out the trash there's a performance cost to it (see [Discord's trouble with Go's GC](https://discord.com/blog/why-discord-is-switching-from-go-to-rust)). Also, memory (de)allocation is slow, so controlling when that happens has have performance benefits.

But, it also allows the programmer to not have to manage memory themselves. Developer time is typically more expensive than CPU time. Also, memory management is _hard_.

---

# Manual Memory Management is Hard

[.column]

One big advantage of garbage collectors is that correctly managing memory allocations is hard. Doing it incorrectly leads to one of the most common classes of security bug: **use after free**. A program can access data on the computer it shouldn't have access to via pointers to freed memory.

[.column]

![inline, right](image.png)

_[source](https://xeiaso.net/shitposts/no-way-to-prevent-this/CVE-2023-6246/)_

---

# Where it gets messy: Pointers

In languages with manual memory management (notably `C++`), **pointers** are a big part of the language. Rather than holding data itself, pointers are the physical memory address[^1] of the underlying data.

Passing pointers around your program is much faster than copying actual data, but you have to ensure your pointers stay valid.

[^1]: Modern computers with virtual memory complicate this explanation a bit, but don't worry about that for now

---

# A Broken C++ Example

C++ lets you initialize a pointer and then delete it, leaving a **dangling pointer**:

```cpp
int *ptr = new int;
*ptr = 5;

cout << *ptr << endl; // 5

delete ptr;

cout << *ptr << endl; // undefined behavior!
```

It's impossible to know what lies on the other end of that dangling pointer. The OS could have put anything (or nothing) there in the meantime. Maybe the program crashes, maybe we read secret data.

---

# Enter: The Borrow Checker

Rust's approach to memory is to allow for using pointers, but statically analyze what you can do with them. Our C++ example doesn't compile when written in Rust:

[.column]

```rust
let s = String::from("cool string!");

let ptr = &s;

println!("{}", *ptr);

drop(s);

println!("{}", *ptr);
```

[.column]

```
  --> src/main.rs:8:10
   |
2  |     let s = String::from("cool string!");
   |         - binding `s` declared here
3  |
4  |     let ptr = &s;
   |               -- borrow of `s` occurs here
...
8  |     drop(s);
   |          ^ move out of `s` occurs here
9  |
10 |     println!("{ptr}");
   |               ----- borrow later used here
```

---

[.build-lists: true]

# The Rules

The core of Rust's memory is its memory ownership model. It follows a three simple rules:

1. Each value in Rust has an owner.
2. There can only be one owner at a time.
3. When the owner goes out of scope, the value will be dropped.

---

# Borrowing isn't Ownership

[.column]

```rust
let s = String::from("cool string!");

let ptr = &s;
let ptr2 = &s;

println!("{}", *ptr);
println!("{}", *ptr2);
```

[.column]

While `s` owns the String data, the pointers `ptr` and `ptr2` only "borrow" it. It's fine to borrow multiple times.

---

# Moving Takes Ownership

[.column]

```rust
let s = String::from("cool string!");

let ptr = &s;

let s2 = s; // moved! s2 is the owner now

println("{}", s); // compiler error
println("{}", &ptr); // compiler error
```

[.column]

A "move" happens when data is assigned to a new variable. Moving invalidates all existing references.

Having two owners would mean it's unclear when memory could be freed - when owner 1 leaves scope or when owner 2 does?

---

## The Borrow Checker:

# The Best of Both Worlds

Rust's Borrow Checker allows for the speed of manual memory management combined with the memory safety of GC languages.

It allows us to eliminate entire classes of bugs, such as data changing out from under you (you have to be explicit about what code is allowed to modify a value).

---

# Barely Touched the Surface

There's a lot that makes Rust a neat language and I only touched on a small, specific piece of it here.

If you're interested in more, the [Rust Book](https://doc.rust-lang.org/book/) is a great (and free!) resource to get your bearings.

---

# We Haven't Won Yet

While Rust is extremely useful in certain situations, it has some significant drawbacks:

- "Fighting the Borrow Checker" is a constant activity - it's hard to do everything exactly right
- Compile time is fairly slow since there's a lot of analysis to be done
- We don't use it at Stripe! So while this is interesting, it won't necessarily change the code we write here.

---

# Any questions?
