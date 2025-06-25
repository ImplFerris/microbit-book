# Play a Tone

In this section, we will build a simple program that plays a sound when a button is pressed. To keep it interesting, we will use both buttons on the micro:bit board. When `Button A` is pressed, we will play a tone with the pitch `A4`. When `Button B` is pressed, the tone will stop. 

You should already be familiar with how buttons work and how to detect button presses from the [Buttons](../smiley-buttons/index.md) chapter. So in this section, we will not go into those details again.


## Create Project from template
For this exercise also, we will be using the microbit-bsp crate with Embassy support.

To generate a new project using the template, run the following command:

```sh
cargo generate --git https://github.com/lulf/embassy-template.git -r f3179dc
```

You will be prompted to enter a project name. 

After that, you will be asked to select the target microcontroller (MCU). From the list, choose:
```
nrf52833
```

## Update Cargo.toml

We will use the microbit-bsp crate. Open the Cargo.toml file and add the following lines:

```toml
# microbit-bsp = "0.3.0"
microbit-bsp = { git = "https://github.com/lulf/microbit-bsp", rev = "9c7d52e" }
```


## Initialization

Let's, first initialize the board. From this board instance, we get access to the `pwm0` peripheral and the built-in speaker. We pass both of these to `SimplePwm`, a helper struct from the embassy-nrf crate that sets up a basic PWM output.

Then, we take the SimplePwm instance and pass it to `PwmSpeaker`, a struct from the microbit-bsp crate. This will let us play tones by giving it a pitch and a duration.

```rust
let board = Microbit::default();
let mut speaker = PwmSpeaker::new(SimplePwm::new_1ch(board.pwm0, board.speaker));
```

## Buttons

Unlike the `microbit-v2` crate, the `microbit-bsp` crate does not group the buttons into a separate buttons struct. Instead, you can access them directly using `board.btn_a` and `board.btn_b`, as shown below:

```rust
let mut button_a = board.btn_a;
let mut button_b = board.btn_b;
```


## Wait...for...it...

Now we will use a loop to keep checking for button presses. Depending on which button is pressed, the playing state will change. If Button A is pressed (i.e. it goes low), it will start playing a tone. It will keep playing until Button B is pressed (i.e. it goes low). 

We will use the wait_for_low() async function to pause the program until the button is pressed, without blocking or wasting CPU power.

```rust
loop {
        button_a.wait_for_low().await;
        speaker.start_note(Pitch::Named(NamedPitch::A4));
        button_b.wait_for_low().await;
        speaker.stop();
    }
```

## The Full code

```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_nrf::pwm::SimplePwm;
use microbit_bsp::{
    Microbit,
    speaker::{NamedPitch, Pitch, PwmSpeaker},
};
use {defmt_rtt as _, panic_probe as _};

#[embassy_executor::main]
async fn main(_spawner: Spawner) -> ! {
    let board = Microbit::default();

    let mut button_a = board.btn_a;
    let mut button_b = board.btn_b;

    let mut speaker = PwmSpeaker::new(SimplePwm::new_1ch(board.pwm0, board.speaker));

    loop {
        button_a.wait_for_low().await;
        speaker.start_note(Pitch::Named(NamedPitch::A4));
        button_b.wait_for_low().await;
        speaker.stop();
    }

}
```

## Clone the existing project
You can also clone (or refer) project I created and navigate to the `bsp-embassy/play-tone` folder.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/bsp-embassy/play-tone
```

## Flash

You can flash the program into the micro:bit and should hear the tone

```sh
cargo flash
```
