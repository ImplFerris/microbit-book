# Print Sound Level with microbit's Microphone in Rust

Let's get started with a simple Rust program where we will print the sound level from the micro:bit's microphone to the system console. This will help us observe how the sound level changes in different environments, like a quiet room or a noisy space.

Later, we'll build a fun project where the micro:bit shows an emoji on the LED matrix when it detects a clap or any sudden sound.


## Create Project from template

For this project, we will be using `microbit-bsp` (with Embassy). To generate a new project using the template, run the following command:

```sh
cargo generate --git https://github.com/ImplFerris/mb2-template.git --rev 3d07b56
```

- When it prompts for a project name, type something like "led-dracula".

- When it prompts whether to use async, select "true".

- When it prompts you to select between "BSP" or "HAL", select the option "BSP".

## Binding interrupt

We've already seen similar code in the temperature sensor and accelerometer chapters. Here, we are binding saadc::InterruptHandler to the SAADC interrupt. 

This tells the system that whenever the SAADC finishes converting a signal from the microphone into a number, the interrupt handler will be notified. In this case, we are using the interrupt handler provided by the embassy-nrf crate, which takes care of handling these events for us in the background.

```rust
bind_interrupts!(struct Irqs {
    SAADC => saadc::InterruptHandler;
});
```
This will give us the unit struct "Irqs" that we will use when setting up the microphone.


## Microphone

To use the microphone, we call Microphone::new() and pass four arguments. These include the SAADC (board.saadc), the microphone input pin (board.microphone), the pin used to enable the microphone (board.micen), and the interrupt unit struct "Irqs" which was created earlier.

```rust
let mut mic = Microphone::new(board.saadc, Irqs, board.microphone, board.micen);
```

Once the microphone is set up, we can call the sound_level() function. This will turn on the microphone, take a sound sample, and return a number between 0 and 255. We can then print this value to the console:

```rust
info!("Sound Level: {}", mic.sound_level().await);
```

## The Full code
```rust
#![no_std]
#![no_main]

use defmt::info;
use embassy_executor::Spawner;
use embassy_nrf::{
    bind_interrupts,
    saadc::{self},
};
use embassy_time::Timer;
use microbit_bsp::{Microbit, mic::Microphone};
use {defmt_rtt as _, panic_probe as _};

bind_interrupts!(struct Irqs {
    SAADC => saadc::InterruptHandler;
});

#[embassy_executor::main]
async fn main(_spawner: Spawner) -> ! {
    let board = Microbit::default();

    let mut mic = Microphone::new(board.saadc, Irqs, board.microphone, board.micen);
    loop {
        info!("Sound Level: {}", mic.sound_level().await);
        Timer::after_millis(100).await;
    }
}
```


## Clone the existing project
You can also clone (or refer) project I created and navigate to the `bsp-embassy/mic-sound-level` folder.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/bsp-embassy/mic-sound-level
```


## Run

Now it's time to test the program. Flash it to your micro:bit using:

```sh
cargo run
```
Once it's running, you should see sound level readings printed to your computer's console.

Try clapping or making noise near the micro:bit. You'll notice the sound level values going up when the environment is loud, and dropping back down when it's quiet.
