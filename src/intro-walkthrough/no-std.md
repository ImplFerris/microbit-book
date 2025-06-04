# no_std Rust environment
When you write a regular Rust program, you get access to the full standard library (std). It gives you features like heap allocation, threads, file systems, and networking. But all these features assume one thing: there's an operating system underneath.

In embedded systems, we usually don't have an operating system. There's no filesystem. No network stack. No heap allocator unless you bring your own. You are running directly on the hardware.

That's where no_std comes in.

By adding this line at the top of your code:
```rust
#![no_std]
```

You tell the Rust compiler: "I don't need the standard library. I'll manage with just the core language features."

Rust will now link only the minimal core crate, which includes the essentials: basic types,  error handling, and so on. This is enough to write logic for most embedded applications.

## Modify code

Let's update our program to include this directive. Open your project in a code editor and modify the `src/main.rs` file as follows:

```rust
#![no_std]
#![no_main]

use cortex_m_rt::entry;

#[entry]
fn main() -> ! {
    loop {}
}
```
