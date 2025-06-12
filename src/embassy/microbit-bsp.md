# Microbit BSP crate that supports Embassy

So far, we have used the `microbit-v2` BSP crate, which operates in blocking mode. Now, let me introduce you to another BSP crate that supports async programming with Embassy: [`microbit-bsp`](https://docs.rs/microbit-bsp/latest/microbit_bsp/). In addition to Embassy integration, this crate includes some handy utilities such as the `scroll` function, which makes it easy to display scrolling text on the LED matrix.

Let's jump in and create a simple async program using this crate.


## Embassy Project Template

Until now, we have been using a custom project template designed specifically for this book. Now, we will use the [Embassy Project Template](https://github.com/lulf/embassy-template), created by Ulf Lilleengen. This template is designed for Embassy-based projects and includes support for a wide range of microcontrollers.  In fact, it was created by the same person who maintains the `microbit-bsp` crate.

To generate a new project using this template, run: 

```sh
#cargo generate --git https://github.com/lulf/embassy-template.git
cargo generate --git https://github.com/lulf/embassy-template.git -r f3179dc
```

> Note: I have included the specific rev (revision) value in the cargo generate command to ensure the setup is reproducible. Without it, future changes to the template might break compatibility with this tutorial.

You will be prompted to enter a project name. 

After that, you will be asked to select the target microcontroller (MCU). From the list, choose:
```
nrf52833
```
This will generate a new project set up with Embassy support for the nrf52833 chip (which is used in the micro:bit v2)

Now you can open the `Cargo.toml` file to see the dependencies included by the template. You will notice Embassy-related crates and async-supported libraries like `embedded-hal-async` already added for you.

## Update Cargo.toml

We will use the microbit-bsp crate, open the `Cargo.toml` and add the following line:

```toml
# microbit-bsp = "0.3.0"
microbit-bsp = { git = "https://github.com/lulf/microbit-bsp", rev = "9c7d52e" }
```

The microbit-bsp crate version 0.3.0 (current version at the time of writing) is not compatible with the embassy project template and will cause dependency conflicts. However, the latest version on GitHub is compatible. To prevent future updates from breaking compatibility with this tutorial, we lock the dependency to a specific rev.


## Prepare main.rs for BSP Code

Open the `src/main.rs` file and remove any existing code inside the main function. We will replace it with code that uses the BSP crate.

## Initialization

We start by creating an instance of the Microbit struct, which gives us access to the board’s peripherals. Then, we retrieve the display from it:

```rust
let board = Microbit::default();
let mut display = board.display;
```

## Adjusting Brightness

The BSP crate provides a nice function to control the brightness of the LED matrix. The brightness value can range from 0 (`Brightness::MIN`) to 10 (`Brightness::MAX`). You can experiment with different values to see how the LED brightness changes.

```rust
display.set_brightness(Brightness::new(5));
```

## Scorlling Text

The BSP crate provides two functions to scroll text across the LED display. The `scroll` function automatically calculates the duration based on the text length, while `scroll_with_speed` gives us full control over the scrolling speed by letting you specify a Duration.

In our examples, we will use scroll_with_speed so that we can control how fast the text scrolls.

```rust
display.scroll_with_speed("EMBASSY", Duration::from_secs(10)).await;
```

## Full Code

Inside the main loop, we continuously scroll the text "EMBASSY" across the display. After each scroll, we add a short delay using `embassy_time::Timer` before repeating.

```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_time::{Duration, Timer};
use microbit_bsp::Microbit;
use microbit_bsp::display::Brightness;
use {defmt_rtt as _, panic_probe as _};

#[embassy_executor::main]
async fn main(_spawner: Spawner) -> ! {
    let board = Microbit::default();
    let mut display = board.display;

    display.set_brightness(Brightness::new(5));

    loop {
        display
            .scroll_with_speed("EMBASSY", Duration::from_secs(10))
            .await;
        Timer::after_secs(1).await;
    }
}
```

## Clone the existing project
You can also clone (or refer) project I created and navigate to the `bsp-embassy/led-scroll` folder.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/bsp-embassy/led-scroll
```

## Flash

You can flash the program into the micro:bit and see the scrolling text. Also, adjust the brightness value and observe.

```sh
cargo flash
```
