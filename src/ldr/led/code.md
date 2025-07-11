# Build a Light-Activated LED Circuit with Microbit and Rust

Let's get started. We did enough theory in the last couple of sections. Time to do some coding. We'll define a threshold for the ADC value. If it goes above that (which should happen when the room gets dark), we'll show the voltage emoji (⚡) on the LED matrix.

## Create Project from template

For this project, we will be using `microbit-bsp` (with Embassy). To generate a new project using the template, run the following command:

```sh
cargo generate --git https://github.com/ImplFerris/mb2-template.git --rev 3d07b56
```

- When it prompts for a project name, type something like "led-dracula".

- When it prompts whether to use async, select "true".

- When it prompts you to select between "BSP" or "HAL", select the option "BSP".

## Initialize Display

We will begin by initializing the micro:bit display. Then, we define a 5x5 flash emoji pattern and set the display brightness to maximum.

```rust
let board = Microbit::default();
let mut display = board.display;

#[rustfmt::skip]
const FLASH: [u8; 5] = [
    0b00010,
    0b00100,
    0b01110,
    0b00100,
    0b01000,
];
display.set_brightness(Brightness::MAX);
```

### Configure ADC 

We will once again use the bind_interrupts! macro to connect the SAADC interrupt to its corresponding handler.

```rust
bind_interrupts!(struct Irqs {
    SAADC => saadc::InterruptHandler;
});
```

Now let's set up the ADC. We'll configure it in single-ended mode, which means it will measure voltage on single pin relative to ground. In this case, we're using pin P0 on the micro:bit, which corresponds to GPIO pin P0.02 on the nRF52833 chip.

```rust
let channel_config = ChannelConfig::single_ended(board.p0);
let config = saadc::Config::default();
let mut adc = Saadc::new(board.saadc, Irqs, config, [channel_config]);
```

### Reading Sample

Now let's read the ADC value. We'll do this in a loop and compare the result against a threshold. The sample function requires a mutable buffer with the same number of elements as the number of channels we configured. Since we’re only using one channel, we provide a 1-element array. The ADC will perform the conversion and store the result in the first (and only) element of the array.
 
```rust
let mut buf = [0; 1];
adc.sample(&mut buf).await;
```

### Displaying Emoji

Once we have the ADC value, controlling the LED display is straightforward. If the value is greater than a certain threshold (in this case, 3500), we show the emoji. Otherwise, we clear the display. You can tweak the threshold to make it more or less sensitive depending on your setup.

```rust
const THRESHOLD: i16 = 3500;
```

```rust
if buf[0] > THRESHOLD {
    display.apply(display::fonts::frame_5x5(&FLASH));
    display.render();
} else {
    display.clear();
}
```rust

```
### Final code

```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_nrf::{
    bind_interrupts,
    saadc::{self, ChannelConfig, Saadc},
};
use microbit_bsp::{
    Microbit,
    display::{self, Brightness},
};

use {defmt_rtt as _, panic_probe as _};

bind_interrupts!(struct Irqs {
    SAADC => saadc::InterruptHandler;
});

const THRESHOLD: i16 = 3500;

#[embassy_executor::main]
async fn main(_spawner: Spawner) -> ! {
    let board = Microbit::default();
    let mut display = board.display;

    let config = saadc::Config::default();
    let channel_config = ChannelConfig::single_ended(board.p0);
    let mut adc = Saadc::new(board.saadc, Irqs, config, [channel_config]);

    #[rustfmt::skip]
    const FLASH: [u8; 5] = [
        0b00010,
        0b00100,
        0b01110,
        0b00100,
        0b01000,
    ];

    display.set_brightness(Brightness::MAX);
    loop {
        let mut buf = [0; 1];
        adc.sample(&mut buf).await;
        if buf[0] > THRESHOLD {
            display.apply(display::fonts::frame_5x5(&FLASH));
            display.render();
        } else {
            display.clear();
        }
    }
}
```

## Clone the existing project

You can also clone (or refer) project I created and navigate to the `ldr-dracula` folder.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/bsp-embassy/ldr-dracula
```

## Flash

All that's left is to flash the code to your micro:bit and see it in action.

Run the following command from your project folder:
```rust
cargo run
```

Now try turning off the room light, moving the LDR to a darker area, or simply covering it with your hand. You should see the LED matrix display the voltage emoji when it gets dark enough. If it doesn't trigger as expected, you can adjust the threshold value to better match your environment.
