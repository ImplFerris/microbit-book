# Fun with LED

Phew... that was a long chapter, and it might have felt a bit overwhelming if you're just getting started. But now, let's take a break and have some fun with the LED matrix!

The BSP gives us a simple and beginner-friendly API. You can just define a 2D array that matches the actual layout of the LED matrix on the micro:bit. Use 1 to turn an LED on and 0 to turn it off. The BSP takes care of everything else behind the scenes.

This is an example LED matrix displaying a heart shape. I tried creating a Ferris (crab) shape, but it didn't quite look right, and I'd have to convince you it was a crab. So, it's better to just show a heart shape instead.

```rust
// LED matrix for heart shape
let led_matrix = [
    [0, 1, 0, 1, 0],
    [1, 1, 1, 1, 1],
    [1, 1, 1, 1, 1],
    [0, 1, 1, 1, 0],
    [0, 0, 1, 0, 0],
];
```

## Display

The BSP provides a `Display` struct that you can initialize using the `display_pins` from the board. It offers a few useful functions, the most important ones being `show` and `clear`. The `show` function takes a 2D array and turns on the LEDs based on the values you provide. If you're curious about how it works under the hood, you can check out the source code [here](https://github.com/nrf-rs/microbit/blob/d64e1b2be8ebd84f73c2df20f91afa8b3c1c949e/microbit-common/src/display/blocking.rs#L154).

## Create Project from template

We will no longer create `.cargo/config.toml`, `memory.x`, or manually add dependencies (only the basic dependencies) every time we start a new project. Instead, we will use a template-based approach to make project setup much easier.

To generate a new project using the template, run the following command:

```sh
cargo generate --git https://github.com/ImplFerris/mb2-template.git
```

When prompted for a project name, enter something like led-matrix.

Once the project is created, update `src/main.rs` with the following code.

## The full code

```rust
#![no_std]
#![no_main]

use embedded_hal::delay::DelayNs;
use microbit::{board::Board, display::blocking::Display, hal::timer::Timer};

use cortex_m_rt::entry;

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

#[entry]
fn main() -> ! {
    let board = Board::take().unwrap();

    let mut timer = Timer::new(board.TIMER0);

    let mut display = Display::new(board.display_pins);
    let led_matrix = [
        [0, 1, 0, 1, 0],
        [1, 1, 1, 1, 1],
        [1, 1, 1, 1, 1],
        [0, 1, 1, 1, 0],
        [0, 0, 1, 0, 0],
    ];
    loop {
        display.show(&mut timer, led_matrix, 1000);
        display.clear();
        timer.delay_ms(1000);
    }
}
```

Everything in this code is pretty straightforward and simple, except for one thing: what is the third argument to `show` function, the one where we pass 1000?

It is called duration_ms, but what is that duration used for? No, it is not for the blinking effect. We already handle blinking separately using:

```rust
display.clear();
timer.delay_ms(1000);
```

So what is duration_ms really doing?

As we learned earlier, the micro:bit's 5x5 LED matrix is multiplexed. Only one row is turned on at a time, and the columns are set based on the image. Internally `show` function goes through each row, lights up the correct LEDs, waits a bit using delay.delay_us(...), then moves to the next row.

This scanning happens so quickly that it looks like the whole image is being displayed at once. The duration_ms value tells the display how long to keep repeating this scan.

In short:
duration_ms(third argument) controls how long the image stays on the screen. You can adjust its value and see the effect.

## Clone the existing project
You can also clone (or refer) project I created and navigate to the `led-matrix` folder.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/bsp/led-matrix
```


## Flash

You can flash the program into the micro:bit and should see the heart shape with blinking effect

```sh
cargo flash
```