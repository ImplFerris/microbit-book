# Abstraction Layers

When working with embedded Rust, you will often come across terms like PAC, HAL, and BSP. These are the different layers that help you interact with the hardware. Each layer offers a different balance between flexibility and ease of use.

Let's start from the highest level of abstraction down to the lowest.

> Note: Throughout this book, we will use whichever crate best fits the needs of each exercise. Some exercises may use a Board Support Package (BSP), while others may rely directly on the Hardware Abstraction Layer (HAL).
 

## Board Support Package (BSP)

A BSP, also referred as Board Support Crate in Rust, tailored to specific development boards.  It combines the HAL with board-specific configurations, providing ready to use interfaces for onboard components like LEDs, buttons, and sensors. This allows developers to focus on application logic instead of dealing with low-level hardware details.  micro:bit also has Board support crate, you can find it [here](https://crates.io/crates/microbit-v2).

In the quick start chapter, we in fact used this.

Example code snippet for BSP

```rust
// Turn on the first LED in the first row
use cortex_m_rt::entry;
use embedded_hal::digital::OutputPin;
use microbit::board::Board;

#[entry]
fn main() -> ! {
    let mut board = Board::take().unwrap();

    let _ = board.display_pins.col1.set_low();
    let mut row1 = board.display_pins.row1;
    let _ = row1.set_high();

    loop {}
}
```

## Hardware Abstraction Layer (HAL)

The HAL sits just below the BSP level. If you work with boards like the Raspberry Pi Pico or ESP32 based boards, you'll mostly use the HAL level. In this book, after some BSP examples, we will focus more on HAL.

The HAL builds on top of the PAC and provides simpler, higher-level interfaces to the microcontroller's peripherals. Instead of handling low-level registers directly, HALs offer methods and traits that make tasks like setting timers, setting up serial communication, or controlling GPIO pins easier.

HALs usually implement the `embedded-hal` traits, which are standard, platform-independent interfaces for peripherals like GPIO, SPI, I2C, and UART. This makes it easier to write drivers and libraries that work across different hardware as long as they use a compatible HAL.

Later, we will explore the `nrf52833-hal`. As you can see, this crate is no longer specific to a dev board but instead tied to the nRF52833 chip. So if another dev board uses the same chip, you can mostly use the same code.


Example code snippet for HAL

```rust
// Turn on the first LED in the first row
use cortex_m_rt::entry;
use embedded_hal::digital::OutputPin;
use nrf52833_hal::gpio::{p0, Level};
use nrf52833_hal::pac::Peripherals;

#[entry]
fn main() -> ! {
    let p = Peripherals::take().unwrap();
    let port0 = p0::Parts::new(p.P0);
    let mut col1 = port0.p0_28.into_push_pull_output(Level::High);
    let mut row1 = port0.p0_21.into_push_pull_output(Level::Low);

    col1.set_low().unwrap();
    row1.set_high().unwrap();

    loop {}
}
```

If you compare this to BSP code, you'll find BSP code easier to read. But at the HAL level, things get more complex. Unless you have some background in embedded programming or electronics, these terms might seem strange. Don't worry; we'll cover all this step by step later.

---
**NOTE:**

The layers below the HAL are rarely used directly. In most cases, the PAC is accessed through the HAL, not on its own. Unless you are working with a chip that does not have a HAL available, there is usually no need to interact with the lower layers directly. In this book, we will focus on the BSP and HAL layers.

---


## Peripheral Access Crate (PAC)

PACs are the lowest level abstraction. They are auto generated crates that give type-safe access to a microcontroller's peripherals. These crates are usually created from the manufacturer's SVD (System View Description) file using tools like `svd2rust`. PACs give you a structured and safe way to work with hardware registers directly.


Example code snippet for PAC

```rust
// Turn on the first LED in the first row

use cortex_m_rt::entry;
use nrf52833_pac::Peripherals;

#[entry]
fn main() -> ! {
    let p = Peripherals::take().unwrap();
    let gpio0 = p.P0;

    gpio0.pin_cnf[21].write(|w| {
        w.dir().output();
        w.input().disconnect();
        w.pull().disabled();
        w.drive().s0s1();
        w.sense().disabled();
        w
    });
    gpio0.pin_cnf[28].write(|w| {
        w.dir().output();
        w.input().disconnect();
        w.pull().disabled();
        w.drive().s0s1();
        w.sense().disabled();
        w
    });

    gpio0.outclr.write(|w| w.pin28().clear());
    gpio0.outset.write(|w| w.pin21().set());

    loop {}
}
```


## Raw MMIO

Raw MMIO (memory-mapped IO) means directly working with hardware registers by reading and writing to specific memory addresses.  This approach mirrors traditional C-style register manipulation and requires the use of `unsafe` blocks in Rust due to the potential risks involved.  We will not touch this area; I haven't seen anyone using this approach, and even if they do, it's outside the scope of this book.


Example code snippet

```rust
// Turn on the first LED in the first row

#![no_main]
#![no_std]

extern crate panic_halt as _;

use nrf52833_pac as _;

use core::mem::size_of;
use cortex_m_rt::entry;

const GPIO_P0: usize = 0x5000_0000;

const PIN_CNF: usize = 0x700;
const OUTSET: usize = 0x508;
const OUTCLR: usize = 0x50c;

const DIR_OUTPUT: u32 = 0x1;
const INPUT_DISCONNECT: u32 = 0x1 << 1;
const PULL_DISABLED: u32 = 0x0 << 2;
const DRIVE_S0S1: u32 = 0x0 << 8;
const SENSE_DISABLED: u32 = 0x0 << 16;

#[entry]
fn main() -> ! {
    let pin_cnf_21 = (GPIO_P0 + PIN_CNF + 21 * size_of::<u32>()) as *mut u32;
    let pin_cnf_28 = (GPIO_P0 + PIN_CNF + 28 * size_of::<u32>()) as *mut u32;
    unsafe {
        pin_cnf_21.write_volatile(
            DIR_OUTPUT | INPUT_DISCONNECT | PULL_DISABLED | DRIVE_S0S1 | SENSE_DISABLED,
        );
        pin_cnf_28.write_volatile(
            DIR_OUTPUT | INPUT_DISCONNECT | PULL_DISABLED | DRIVE_S0S1 | SENSE_DISABLED,
        );
    }

    let gpio0_outset = (GPIO_P0 + OUTSET) as *mut u32;
    let gpio0_outclr = (GPIO_P0 + OUTCLR) as *mut u32;
    unsafe {
        gpio0_outclr.write_volatile(1 << 28);
        gpio0_outset.write_volatile(1 << 21);
    }

    loop {}
}
```

## Reference

- The code snippets of PAC and Raw MMIO is taken from the Google's [Comprehensive Rust book](https://comprehensive-rust.pages.dev/bare-metal/microcontrollers/mmio)
