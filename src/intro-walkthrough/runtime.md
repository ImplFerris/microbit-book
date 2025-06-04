# Runtime 

Let's start with the first dependency `cortex-m-rt`. This crate provides the startup code and a minimal runtime for Cortex-M microcontrollers. As you may already know, our micro:bit is also based on a Cortex-M core.

## Why do we need?
In embedded development, there is typically no underlying OS (though specialized operating systems for microcontrollers do exist). This means you must set up everything yourself: how the program starts, how memory is initialized, and how the device responds to events like button presses or incoming data.

To make all of this work, we will use a runtime crate. A runtime in embedded Rust provides the minimal startup code that runs before your main function, sets up memory (like the stack and heap), and helps you define how your program should react to [interrupts](../core-concepts/interrupts.md).


---

## Entry point

From a developer's perspective, it might seem like the `main` function is the first code executed when a program runs. However, this isn't actually the case. In most languages, a runtime system sets up the environment before eventually calling main.

In contrast, embedded systems like the micro:bit don't have a standard runtime. Instead, we use a custom runtime, such as the one provided by the cortex-m-rt crate. In this setup, we need to explicitly specify the program's entry point. For the micro:bit v2, it is done using the `#[entry]` [attribute](https://docs.rs/cortex-m-rt/latest/cortex_m_rt/attr.entry.html) provided by cortex-m-rt, which tells the runtime which function to run first.

## no_main

We will use the `#![no_main]` directive to tell the Rust compiler that we don't want to use the normal program entry point. Instead, we will provide our own entry point and main function. 

By adding #![no_main], we disable the default program's startup logic, allowing the embedded runtime (like cortex-m-rt) to take control.


## Modify code

Let's update our program to include these attributes. Open your project in a code editor and modify the `src/main.rs` file as follows:

```rust
#![no_std]
#![no_main]

use cortex_m_rt::entry;

#[entry]
fn main() {
    println!("Hello, world!");
}
```

After you update the code, you will likely see an error from rust-analyzer saying:

```rust
error: `#[entry]` function must have signature `[unsafe] fn() -> !`
```

This happens because the `#[entry]` function is required to never return. It must have the return type `!` (called the "never" type) to show that the program runs indefinitely and does not exit. This requirement is critical in embedded or bare-metal systems because, unlike applications running on a traditional operating system, there is no OS to hand control back to once the program finishes. 

To fix this, update your main function signature to:

```rust
#[entry]
fn main() -> ! {
    loop {
        // keep running forever
    }
}
```

If you have Clippy enabled, you might see a warning saying "empty loop {} wastes CPU cycles." You can safely ignore this warning for now.


## References:

- cortex_m_rt crate document: [https://docs.rs/cortex-m-rt/latest/cortex_m_rt/index.html](https://docs.rs/cortex-m-rt/latest/cortex_m_rt/index.html)

- A Freestanding Rust Binary - Start attribute: [https://os.phil-opp.com/freestanding-rust-binary/#the-start-attribute](https://os.phil-opp.com/freestanding-rust-binary/#the-start-attribute)

