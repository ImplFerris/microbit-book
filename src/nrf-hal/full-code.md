# Putting It All Together: Using a nrf-hal to Blink an LED

After setting up the peripherals, initializing the timer, and configuring the required GPIO pin as output, the main loop becomes straightforward. It closely resembles what we did in the BSP example.

In the loop, we toggle the state of row1 between high and low; with a delay from the timer. This creates the blinking effect.

```rust
    loop {
        timer0.delay_ms(500);
        row1.set_high().unwrap();

        timer0.delay_ms(500);
        row1.set_low().unwrap();
    }
```

## Full code
```rust
#![no_std]
#![no_main]

use cortex_m_rt::entry;

// Embedded HAL traits
use embedded_hal::delay::DelayNs;
use embedded_hal::digital::OutputPin;

// nRF HAL
use nrf52833_hal::gpio::Level;
use nrf52833_hal::pac::Peripherals;
use nrf52833_hal::{self as hal, timer::Timer};

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

#[entry]
fn main() -> ! {
    let peripherals = Peripherals::take().unwrap();
    let mut timer0 = Timer::new(peripherals.TIMER0);

    let port0 = hal::gpio::p0::Parts::new(peripherals.P0);
    let _col1 = port0.p0_28.into_push_pull_output(Level::Low);
    let mut row1 = port0.p0_21.into_push_pull_output(Level::High);

    loop {
        timer0.delay_ms(500);
        row1.set_high().unwrap();

        timer0.delay_ms(500);
        row1.set_low().unwrap();
    }
}

```


## Clone the existing project
You can also clone (or refer) project I created and navigate to the `hal/led-blinky` folder.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/hal/led-blinky
```


## Flash

You can flash the program into the micro:bit and should see the blinking effect

```sh
cargo flash
```
