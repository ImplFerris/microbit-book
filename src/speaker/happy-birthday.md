# Writing Rust Code to Play "Happy Birthday" on micro:bit v2

In this section, we will play the "Happy Birthday" tune on the microbit speaker. 

The functions provided by the microbit-bsp crate is great for micro:bit, but I wanted something that can be used across different MCUs like ESP32 or Raspberry Pi Pico. So I made a separate crate that defines notes and durations more cleanly, using musical terms like Quarter, Half, etc. In the next section, we'll start using that crate to play tunes in a more reusable and portable way.

For this, we will use a crate called `tinytones`. It comes with the "Happy Birthday" tune built in, so we do not need to define the pitches and durations ourselves. 

The crate also provides a `Pitch` enum and a `Tone` struct for defining your own melodies. It includes helper functions to convert musical durations like Quarter or Half into actual time values based on the tempo of the song.

## Create Project from template

For this project, we will be using `microbit-bsp` (with Embassy). To generate a new project using the template, run the following command:

```sh
cargo generate --git https://github.com/ImplFerris/mb2-template.git --rev 3d07b56
```

- When it prompts for a project name, type something like "play-song".

- When it prompts whether to use async, select "true".

- When it prompts you to select between "BSP" or "HAL", select the option "BSP".

## Update Cargo.toml

We will add tinytones crate. Open the Cargo.toml file and add the following lines:

```toml
tinytones = { version="0.1.0" }
```

## Imports
These are the imports needed for this program. Open the `main.rs` file and update it with the following:

```rust
// Default that comes with the Template
use embassy_executor::Spawner;
use embassy_time::Timer;
use {defmt_rtt as _, panic_probe as _};

// Additional Import
use embassy_nrf::pwm::SimplePwm;
use microbit_bsp::{
    Microbit,
    speaker::{Note, Pitch, PwmSpeaker},
};
use tinytones::{Tone, songs};

```

## Initialization

Let's, first initialize the board. From this board instance, we get access to the `pwm0` peripheral and the built-in speaker. We pass both of these to `SimplePwm`, a helper struct from the embassy-nrf crate that sets up a basic PWM output.

Then, we take the SimplePwm instance and pass it to `PwmSpeaker`, a struct from the microbit-bsp crate. This will let us play tones by giving it a pitch and a duration.

```rust
let board = Microbit::default();
let mut speaker = PwmSpeaker::new(SimplePwm::new_1ch(board.pwm0, board.speaker));
```

## Choose a Tune

The tinytones crate provides a set of built-in songs and tunes; You can browse the full list in the [documentation](https://docs.rs/tinytones/latest/tinytones/songs/index.html). 

In this example, we will be playing the "Happy Birthday" tune. To do that, we initialize a Tone struct by calling Tone::new. The first argument is the tempo (how fast or slow the song plays) of the song. You can either use the predefined tempo provided with the tune or specify your own (e.g: 150). The second argument is the melody, which is a list of notes (each with a pitch and duration). 

```rust
let song = Tone::new(songs::happy_birthday::TEMPO, songs::happy_birthday::MELODY);
```

## Play the Tune in a Loop
Once we have the song loaded, we can play it. The `Tone` provides an iter() method, which gives us an iterator over each note in the melody. Each note is a pair of (pitch, duration).

Inside the loop, we go through each note one by one. If the pitch is `Rest`, that means it's a silent pause. In that case, we just wait for the note's duration using Timer::after_millis.

For all other notes, we play the sound using the speaker.play() method. This takes a Note, which we create by converting the pitch into a frequency using pitch.freq_u32(), and then passing in the note's duration.

```rust
loop {
    for (pitch, note_duration) in song.iter() {
        if pitch == tinytones::note::Pitch::Rest {
            Timer::after_millis(note_duration).await;
            continue;
        }

        speaker
            .play(&Note(
                Pitch::Frequency(pitch.freq_u32()),
                note_duration as u32,
            ))
            .await;
    }
    Timer::after_secs(5).await;
}
```

After finishing the whole tune, we wait for 5 seconds before looping again and playing the song from the beginning.


## The Full code
```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_nrf::pwm::SimplePwm;
use embassy_time::Timer;
use microbit_bsp::{
    Microbit,
    speaker::{Note, Pitch, PwmSpeaker},
};
use tinytones::{Tone, songs};
use {defmt_rtt as _, panic_probe as _};

#[embassy_executor::main]
async fn main(_spawner: Spawner) -> ! {
    let board = Microbit::default();
    let mut speaker = PwmSpeaker::new(SimplePwm::new_1ch(board.pwm0, board.speaker));

    let song = Tone::new(songs::happy_birthday::TEMPO, songs::happy_birthday::MELODY);
    loop {
        for (pitch, note_duration) in song.iter() {
            if pitch == tinytones::note::Pitch::Rest {
                Timer::after_millis(note_duration).await;
                continue;
            }

            speaker
                .play(&Note(
                    Pitch::Frequency(pitch.freq_u32()),
                    note_duration as u32,
                ))
                .await;
        }
        Timer::after_secs(5).await;
    }
}
```


## Clone the existing project
You can also clone (or refer) project I created and navigate to the `bsp-embassy/play-song` folder.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/bsp-embassy/play-song
```


## Flash

You can flash the program into the micro:bit and should hear the melody

```sh
cargo flash
```
