
# Panic Handler

I assume you already have a basic idea of what a panic is in Rust. When a panic happens, the program does not immediately exit. Instead, control is passed to the panic handler provided by the standard lib. By default, it starts unwinding the stack of the panicking thread. But if the user has chosen to abort on panic, then the program will terminate right away without unwinding.

If we try to build the program at this stage, we will see this error:

```sh
error: `#[panic_handler]` function required, but not found

error: unwinding panics are not supported without std
```

This means we need to define a custom panic handler, since we're not using the standard library.


## Panic handler in no_std

In a `no_std` environment, we have to provide our own panic handler. There are crates that do this for us, and we can pick one based on the behavior we want.

For example:

- If we want the program to abort immediately, we can use the `panic-abort` crate.

- If we want the program (or the current thread) to halt by entering an infinite loop, we can use the `panic-halt` crate.

If you check the source code of these crates, you will notice they are just simple functions.

You can either import one of these crates like this:

```rust
use panic_halt as _;
```

Or define the panic handler function yourself. That's what we're going to do.

The function must be marked with the `#[panic_handler]` attribute, and it must accept a reference to core::panic::PanicInfo. It should never return, so its return type is !.

Here's the function (equivalent to what panic-halt provides). Let's update the `src/main.rs` file and include the following code

```rust
#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}
```



## Reference

- [Panicking section](https://docs.rust-embedded.org/book/start/panicking.html) in The Embedded Rust book 

- [panic_handler section](https://doc.rust-lang.org/nomicon/panic-handler.html) in The Rustonomicaon

- [Bare-metal target for CPUs in the Armv7E-M architecture family](https://doc.rust-lang.org/nightly/rustc/platform-support/thumbv7em-none-eabi.html)