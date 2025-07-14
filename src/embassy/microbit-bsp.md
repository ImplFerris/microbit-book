# Microbit BSP crate that supports Embassy

So far, we have used the `microbit-v2` BSP crate, which operates in blocking mode. Now, let me introduce you to another BSP crate that supports async programming with Embassy: [`microbit-bsp`](https://docs.rs/microbit-bsp/latest/microbit_bsp/). In addition to Embassy integration, this crate includes some handy utilities such as the `scroll` function, which makes it easy to display scrolling text on the LED matrix.

Let's jump in and create a simple async program using this crate.


## Embassy Project Template

So far, we have been using a custom project template designed specifically for this book. You can also use the [Embassy Project Template](https://github.com/lulf/embassy-template), created by Ulf Lilleengen. This template is designed for Embassy-based projects and includes support for a wide range of microcontrollers.  In fact, it was created by the same person who maintains the `microbit-bsp` crate.

```sh
cargo generate --git https://github.com/lulf/embassy-template.git
```

When prompted to select the target microcontroller, choose "nrf52833". This will create a new project configured with Embassy support for the nrf52833 chip (which powers the micro:bit v2).

Originally, I was using this template to generate Embassy projects. But at the time of writing, it did not have the latest GitHub revision of embassy-nrf. I wanted to use some of the new features in both embassy-nrf and microbit-bsp, so I switched to a custom template.

Still, I kept this here because it is a nice and useful template. It will be helpful once you finish this book and want to explore more.

## Create Project from template

For this project, we will be using `microbit-bsp` (with Embassy). To generate a new project using the template, run the following command:

```sh
cargo generate --git https://github.com/ImplFerris/mb2-template.git --rev 3d07b56
```

- When it prompts for a project name, type something like "led-scroll".

- When it prompts whether to use async, select "true".

- When it prompts you to select between "BSP" or "HAL", select the option "BSP".

Once the project is generated, open the Cargo.toml file. You will see that it includes the microbit-bsp crate along with other Embassy-related crates.

## BSP Boilerplate code

Open the src/main.rs file. You will see some boilerplate code that creates an instance of the Microbit struct. This gives us access to the board's peripherals.

```rust
#[embassy_executor::main]
async fn main(_spawner: Spawner) -> ! {
    let board = Microbit::default();

    loop {
        Timer::after_secs(1).await;
    }
}
```

The template includes a simple loop with a 1 second delay using Timer. We will remove this and write our own loop logic for this project.

## Initialize Display
To use the LED matrix display, we first need to take ownership of it from the board:

```rust
let mut display = board.display;
```
This gives us access to the built-in 5x5 LED display, so we can start showing patterns or animations on it.

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
