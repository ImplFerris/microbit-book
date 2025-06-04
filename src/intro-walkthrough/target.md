## Target

Let's try building it. Wait, it still doesn't build and shows this error:

```sh
rustc: found duplicate lang item `panic_impl`
the lang item is first defined in crate `std` (which `test` depends on)
```

The solution is simple: you just need to specify the target platform explicitly. When you don't specify the target, the compiler builds for your host machine by default, which includes its own panic handler from std. This causes a conflict because your code also provides a panic handler, leading to the duplicate lang item error.

Since our micro:bit uses an ARM Cortex-M4 32-bit processor with a Floating Point Unit (FPU), the correct target to use is thumbv7em-none-eabihf. We already added this target in the quickstart section.

So we can simply build it using the following command:

```sh
cargo build --target thumbv7em-none-eabihf
```

## .cargo/config.toml

There are two things to fix. First, the code editor might still highlight the panic function and show a duplicate error. Second, it's not convenient to type the target every time we build.

To solve this, we can create a config.toml file inside the .cargo directory and set the default target there.

Run the following commands from the root of your project (the same directory where Cargo.toml is located) to create the .cargo directory. Then create the config.toml inside the .cargo directory.

```sh
mkdir .cargo
cd .cargo
```

Update the config.toml with the following content:
```toml
[build]
target = "thumbv7em-none-eabihf"
```

Now try running the build command without specifying the target. It should compile successfully:
```sh
cargo build
```

---

Let's take a look at the `src/main.rs` code we have up to this point.

```rust
#![no_std]
#![no_main]

use cortex_m_rt::entry;

use embedded_hal::{delay::DelayNs, digital::OutputPin};
use microbit::{board::Board, hal::timer::Timer};

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

#[entry]
fn main() -> ! {
    let mut board = Board::take().unwrap();
    let mut timer = Timer::new(board.TIMER0);

    let _ = board.display_pins.col1.set_low();
    let mut row1 = board.display_pins.row1;

    loop {
        let _ = row1.set_low();
        timer.delay_ms(500);
        let _ = row1.set_high();
        timer.delay_ms(500);
    }
}
```