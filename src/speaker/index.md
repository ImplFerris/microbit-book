# Speaker

The micro:bit v2 comes with a built-in speaker, which means we can play sounds, tones, and even simple melodies without connecting any extra hardware. The speaker is wired internally to one of the GPIO pins, and by sending the right signals through software, we can make it produce different pitches.

## Using the microbit-bsp API

The `microbit-bsp` crate makes it easy to use the speaker. It gives us a high-level API that handles all the low-level details for us. Behind the scenes, it uses something called PWM (Pulse-Width Modulation) to generate tones. Don't worry about what PWM is just yet; we'll get into that in a later chapter. For now, we'll just use the API.

## Example Code 

This example is taken from the official [GitHub repository](https://github.com/lulf/microbit-bsp/blob/main/examples/speaker/src/main.rs) of the microbit-bsp crate. It plays a simple tune using the built-in speaker. The crate provides a helper enum like NamedPitch, where each variant represents a musical note. This makes it easy to define your own custom tunes using familiar note names.

```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_time::Timer;
use microbit_bsp::{
    embassy_nrf::pwm::SimplePwm,
    speaker::{NamedPitch, Note, PwmSpeaker},
    Microbit,
};
use {defmt_rtt as _, panic_probe as _};

const TUNE: [(NamedPitch, u32); 18] = {
    #[allow(clippy::enum_glob_use)]
    use NamedPitch::*;
    [
        (D4, 1),
        (DS4, 1),
        (E4, 1),
        (C5, 2),
        (E4, 1),
        (C5, 2),
        (E4, 1),
        (C5, 3),
        (C4, 1),
        (D4, 1),
        (DS4, 1),
        (E4, 1),
        (C4, 1),
        (D4, 1),
        (E4, 2),
        (B4, 1),
        (D5, 2),
        (C4, 4),
    ]
};

#[embassy_executor::main]
async fn main(_s: Spawner) {
    let board = Microbit::default();
    defmt::info!("Application started!");
    let mut speaker = PwmSpeaker::new(SimplePwm::new_1ch(board.pwm0, board.speaker));
    loop {
        defmt::info!("Playing tune!");
        for (pitch, ticks) in TUNE {
            speaker.play(&Note(pitch.into(), 200 * ticks)).await;
        }
        Timer::after_secs(5).await;
    }
}
```

Here, it loops through each pitch defined in the TUNE array and plays it for a given number of ticks. The duration is just ticks * 200 milliseconds. So if a note has ticks = 2, it plays for 400 ms.

