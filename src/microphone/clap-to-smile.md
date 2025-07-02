# Simple Rust Embedded Project: Show a Smiley on microbit When You Clap

In a previous example, we explored how to read the sound level from the micro:bit's built-in microphone and printed those values to the system console. Now, we will take it a step further. In this simple Rust project, we will use sound level detection to recognize a clap and display a smiley face on the micro:bit's 5x5 LED matrix. 

## Create Project

The steps are same as the previous section to print values. Instead of going again with same steps, i suggest you to clone the project and rename something like "clap2smile".

Or You can also clone project I created and work on top of it.

```sh
git clone https://github.com/ImplFerris/microbit-projects
# Copy to your preferred directory
cp microbit-projects/bsp-embassy/mic-sound-level clap2smile
cd clap2smile 
```


## Smiley face

When we were using the microbit-v2 crate, we used a simple 2D matrix to represent the smiley. For the microbit-bsp crate, we will have to define our shape using the Frame and Bitmap structs. 

Each row of the 5x5 LED matrix is represented as a binary number (u8) using the Bitmap::new function. 

```rust
let smile_frame = Frame::<5, 5>::new([
    Bitmap::new(0b00000, 5),
    Bitmap::new(0b01010, 5),
    Bitmap::new(0b00000, 5),
    Bitmap::new(0b10001, 5),
    Bitmap::new(0b01110, 5),
]);

led_matrix.set_brightness(Brightness::MAX);
```

## Threshold
To detect a clap, we need to define a sound level threshold. The micro:bit's microphone provides a value between 0 and 255, where higher values indicate louder sounds. A clap typically causes a sharp spike in this value.

```rust
const CLAP_THRESHOLD: u8 = 180;
```

We can start with a threshold value around 180-200.  You can tweak this value depending on your preference(how loud you want the clap to be) and surroundings.

And the logic is simple. If the sound level exceeds this threshold, we'll consider it a clap and trigger the smiley face display.

```rust
if mic.sound_level().await > CLAP_THRESHOLD {
    led_matrix
        .display(smile_frame, Duration::from_secs(1))
        .await;
}
```

## The full code

```rust
#![no_std]
#![no_main]

// use defmt::info;
use embassy_executor::Spawner;
use embassy_nrf::{
    bind_interrupts,
    saadc::{self},
};
use embassy_time::{Duration, Timer};
use microbit_bsp::{
    Microbit,
    display::{Bitmap, Brightness, Frame},
    mic::Microphone,
};
use {defmt_rtt as _, panic_probe as _};

bind_interrupts!(struct Irqs {
    SAADC => saadc::InterruptHandler;
});

const CLAP_THRESHOLD: u8 = 180;

#[embassy_executor::main]
async fn main(_spawner: Spawner) -> ! {
    let board = Microbit::default();
    let mut led_matrix = board.display;

    let mut mic = Microphone::new(board.saadc, Irqs, board.microphone, board.micen);

    let smile_frame = Frame::<5, 5>::new([
        Bitmap::new(0b00000, 5),
        Bitmap::new(0b01010, 5),
        Bitmap::new(0b00000, 5),
        Bitmap::new(0b10001, 5),
        Bitmap::new(0b01110, 5),
    ]);

    led_matrix.set_brightness(Brightness::MAX);

    loop {
        // info!("Sound Level: {}", mic.sound_level().await);
        if mic.sound_level().await > CLAP_THRESHOLD {
            led_matrix
                .display(smile_frame, Duration::from_secs(1))
                .await;
        }
        Timer::after(Duration::from_millis(100)).await;
    }
}
```


## Clone the existing project
You can also clone (or refer) project I created and navigate to the `bsp-embassy/clap2smile` folder.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/bsp-embassy/clap2smile
```


## Run

Now it's time to test the program. Flash it to your micro:bit using:

```sh
cargo run
```

Once it's running, try clapping near the micro:bit. You should see the smiley face appear on the LED matrix. Btw, it's not specifically detecting a clap - it simply checks if the sound level crosses the threshold. So any loud sound, like a shouting, will also trigger the smiley.
