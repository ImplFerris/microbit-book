# Quick Start

Before diving into the theory and concepts of how everything works, let's jump straight into action. Use this simple code to create blink effect on the Display Matrix of the microbit.


## The Full code

```rust
#![no_std]
#![no_main]

use embedded_hal::{delay::DelayNs, digital::OutputPin};
use microbit::{board::Board, hal::timer::Timer};

use cortex_m_rt::entry;

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

## Clone the Quick start project
You can clone the quick start project I created and navigate to the project folder and run it.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/bsp/blinky
```

## How to Run?

You refer the ["Running The Program"](../running.md) section
