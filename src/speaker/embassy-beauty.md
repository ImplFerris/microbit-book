# The Beauty of Embassy

In this section, we will extend the playing tone exercise by introducing a background task. This demonstrates the er of Embassy's async task model.

The main task will scroll the text "EMBASSY" continuously on the display. At the same time, a background task will wait for button presses. Depending on which button is pressed, it will either start or stop the tone playback, just like in the previous example.

While it is possible to achieve the same result without Embassy, doing so would require much more effort and complexity. Embassy simplifies embedded development.

You can modify the previous exercise or create project from scratch. 


## Create Project from template
For this exercise also, we will be using the microbit-bsp crate with Embassy support.

To generate a new project using the template, run the following command:

```sh
#cargo generate --git https://github.com/lulf/embassy-template.git
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

As usual, we begin by initializing the board, the display, and the speaker.

```rust
let board = Microbit::default();
let mut display = board.display;
let speaker = PwmSpeaker::new(SimplePwm::new_1ch(board.pwm0, board.speaker)); // We are not marking Speaker mut here
```


## Using the Spawner Argument

Until now, we have not used the Spawner argument passed to the main function, nor have we discussed its purpose. In this section, we will start using it. So, remove the underscore from the argument name to make it usable.

```rust
// async fn main(_spawner: Spawner) -> ! {
// To:
async fn main(spawner: Spawner) -> ! {
```

The Spawner allows you to launch background tasks. We will use it to start the button_task, which we will define shortly.

## Button Task
Now let's define the background task that will handle button presses. This is a simple async task marked with the `#[embassy_executor::task]` attribute. It takes ownership of both buttons and the speaker.

In the loop, we wait for Button A to be pressed (goes low), and when that happens, we start playing a note. Then we wait for Button B to be pressed, and when it is, we stop the note.

```rust

#[embassy_executor::task]
async fn button_task(
    mut button_a: Input<'static>,
    mut button_b: Input<'static>,
    mut speaker: PwmSpeaker<'static, PWM0>,
) {
    loop {
        button_a.wait_for_low().await;
        speaker.start_note(Pitch::Named(NamedPitch::A4));

        button_b.wait_for_low().await;
        speaker.stop();
    }
}
```

## Launch the Task

Now that we have defined the button_task, we can launch it from the main function using the Spawner.
```rust
 spawner
        .spawn(button_task(board.btn_a, board.btn_b, speaker))
        .unwrap();
```
That's it. With just this line, the button task starts running in the background, waiting for button presses while the main task continues doing its own work.


## Main Task loop

The main task is simple. It keeps scrolling the text "EMBASSY" on the display in a loop. After each scroll, we add a small delay using a timer.

```rust
 loop {
        display
            .scroll_with_speed("EMBASSY", Duration::from_secs(10))
            .await;
        Timer::after_millis(300).await;
    }
```
While this loop runs continuously, the button task we spawned earlier runs in the background without blocking the main task. This is the beauty of async with Embassy.


## The Full code
```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_nrf::{gpio::Input, peripherals::PWM0, pwm::SimplePwm};
use embassy_time::{Duration, Timer};
use microbit_bsp::{
    Microbit,
    speaker::{NamedPitch, Pitch, PwmSpeaker},
};
use {defmt_rtt as _, panic_probe as _};

#[embassy_executor::main]
async fn main(spawner: Spawner) -> ! {
    let board = Microbit::default();
    let mut display = board.display;

    let speaker = PwmSpeaker::new(SimplePwm::new_1ch(board.pwm0, board.speaker));

    spawner
        .spawn(button_task(board.btn_a, board.btn_b, speaker))
        .unwrap();

    loop {
        display
            .scroll_with_speed("EMBASSY", Duration::from_secs(10))
            .await;
        Timer::after_millis(300).await;
    }
}

#[embassy_executor::task]
async fn button_task(
    mut button_a: Input<'static>,
    mut button_b: Input<'static>,
    mut speaker: PwmSpeaker<'static, PWM0>,
) {
    loop {
        button_a.wait_for_low().await;
        speaker.start_note(Pitch::Named(NamedPitch::A4));

        button_b.wait_for_low().await;
        speaker.stop();
    }
}
```


## Clone the existing project
You can also clone (or refer) project I created and navigate to the `bsp-embassy/background-tasks` folder.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/bsp-embassy/background-tasks
```

## Flash

Now you can flash the program to the micro:bit:

```sh
cargo flash
```

Once flashed, the text "EMBASSY" will scroll continuously on the display. You can press the buttons to start or stop the tune, and it will work smoothly in the background without interrupting the scrolling.
