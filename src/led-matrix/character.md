# Display Character

We were able to display a heart shape on the LED matrix. Now let's take it a step further and try showing some characters on it. 

Let's start by displaying the character 'R'.

```rust
// Matrix for 'R'
[
    [1, 1, 1, 0, 0],
    [1, 0, 0, 1, 0],
    [1, 1, 1, 0, 0],
    [1, 0, 1, 0, 0],
    [1, 0, 0, 1, 0],
],
```


## Create Project from template

To generate a new project using the template, run the following command:

```sh
cargo generate --git https://github.com/ImplFerris/mb2-template.git
```

When prompted for a project name, enter something like led-char.

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
        [1, 1, 1, 0, 0],
        [1, 0, 0, 1, 0],
        [1, 1, 1, 0, 0],
        [1, 0, 1, 0, 0],
        [1, 0, 0, 1, 0],
    ];
    loop {
        display.show(&mut timer, led_matrix, 1000);
        display.clear();
        timer.delay_ms(1000);
    }
}
```


## Clone the existing project
You can also clone (or refer) project I created and navigate to the `led-char` folder.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/bsp/led-char
```


## Flash

You can flash the program into the micro:bit and should see the character

```sh
cargo flash
```