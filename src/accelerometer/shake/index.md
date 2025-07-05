# Writing Shake Detection Code with microbit in Rust

We have successfully printed the accelerometer readings to the system console. We assume you have tried tilting the microbit in different directions and noticed how the values change. While that is fun, simply printing values is not very exciting. In this chapter, we will write a program that plays a tone when you shake the micro:bit.

## Create Project from template
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

Open the Cargo.toml file and add the following lines:

```toml
microbit-bsp = { git = "https://github.com/lulf/microbit-bsp", rev = "9c7d52e" }
lsm303agr = { version = "1.1.0", features = ["async"] }
```

## How to detect shake?

To detect a shake, we first need to calculate the magnitude of the accelerometer readings. The formula for computing the magnitude is:

$$
\text{magnitude} = \sqrt{x^2 + y^2 + z^2}
$$

### Example Calculation

Let's take one sample reading and calculate its magnitude.

Sample value: x = 81, y = -105, z = -1018

```rust
x = 81          => x^2 = 6561
y = -105        => y^2 = 11025
z = -1018       => z^2 = 1036324

magnitude^2 = 6561 + 11025 + 1036324 = 1053910
magnitude   = sqrt(1053910) ≈ 1026.6
```

Let's take one more sample reading: x:-875, y:-567, z:-143
```rust
x^2          = (-875)^2   = 765625
y^2          = (-567)^2   = 321489
z^2          = (-143)^2   = 20449

magnitude^2  = 765625 + 321489 + 20449 = 1107563
magnitude    = sqrt(1107563) ≈ 1052.4
```
### Readings While Shaking the Microbit

You can try a few more sample readings and calculate their magnitudes. Most values at rest should be close to 1000. Now, run the previous program to print accelerometer values. This time, shake the micro:bit well and observe the readings.

Here is one sample I recorded while shaking the device: x:-1265, y:-2029, z:1657

```rust
x^2          = (-1265)^2  = 1600225
y^2          = (-2029)^2  = 4116841
z^2          = (1657)^2   = 2745649

magnitude^2  = 1600225 + 4116841 + 2745649 = 8462715
magnitude    = sqrt(8462715) ≈ 2909.07
```

Woah, now the magnitude is nearly 3000! This is a significant jump from the resting value and clearly indicates a strong shake.

To detect a shake in code, we can choose a threshold value (for example, 2000) and compare the calculated magnitude against it. If the magnitude is greater than this threshold, we can say a shake has occurred.

## How to calculate Square Root(sqrt) in Embedded Rust (#![no_std])

The sqrt() function is not available in a no_std environment by default. While crates like "libm" provide math functions for no_std, we will use a simpler and more efficient approach.

Instead of calculating the square root, we can square the threshold and compare it directly with the squared magnitude. For example, if our threshold is 2000, we square it to get 4,000,000. Then we compare the squared magnitude against this value.

```rust
// Instead of this:
2909.07 > 2000

// We do this:
8462715 > 4000000
```

## Code to detect Shake

Let's convert the threshold comparison logic into Rust code. Below is a function that takes x, y, and z accelerometer readings and returns true if a shake is detected.

```rust
const SHAKE_THRESHOLD_MG: i32 = 2000;
const SHAKE_THRESHOLD_SQUARED: i64 = (SHAKE_THRESHOLD_MG * SHAKE_THRESHOLD_MG) as i64;

fn detect_shake(x: i32, y: i32, z: i32) -> bool {
    let mag_sq = x as i64 * x as i64 + y as i64 * y as i64 + z as i64 * z as i64;
    mag_sq > SHAKE_THRESHOLD_SQUARED
}
```

## The Full code

You may need to adjust the threshold value and delay timing depending on how you shake the device and the level of sensitivity you want to achieve.

```rust
#![no_std]
#![no_main]

use defmt::info;
use embassy_nrf::{self as hal, pwm::SimplePwm, twim::Twim};
use hal::twim;

use defmt_rtt as _;
use embassy_executor::Spawner;
use embassy_time::{Delay, Timer};
use lsm303agr::{AccelMode, AccelOutputDataRate, Lsm303agr};
use microbit_bsp::{
    Microbit,
    speaker::{NamedPitch, Pitch, PwmSpeaker},
};

#[panic_handler]
fn panic(panic_info: &core::panic::PanicInfo) -> ! {
    info!("{:?}", panic_info);
    loop {}
}

hal::bind_interrupts!(struct Irqs {
    TWISPI0 => twim::InterruptHandler<hal::peripherals::TWISPI0>;
});

const SHAKE_THRESHOLD_MG: i32 = 2000;

const SHAKE_THRESHOLD_SQUARED: i64 = (SHAKE_THRESHOLD_MG * SHAKE_THRESHOLD_MG) as i64;

fn detect_shake(x: i32, y: i32, z: i32) -> bool {
    let mag_sq = x as i64 * x as i64 + y as i64 * y as i64 + z as i64 * z as i64;
    mag_sq > SHAKE_THRESHOLD_SQUARED
}

#[embassy_executor::main]
async fn main(_spawner: Spawner) -> ! {
    // let p = embassy_nrf::init(Default::default());
    let board = Microbit::default();

    let twim_config = twim::Config::default();
    let twim0 = Twim::new(
        board.twispi0,
        Irqs,
        board.i2c_int_sda,
        board.i2c_int_scl,
        twim_config,
    );

    let mut speaker = PwmSpeaker::new(SimplePwm::new_1ch(board.pwm0, board.speaker));

    let mut sensor = Lsm303agr::new_with_i2c(twim0);
    sensor.init().await.unwrap();
    sensor
        .set_accel_mode_and_odr(
            &mut Delay,
            AccelMode::HighResolution,
            AccelOutputDataRate::Hz50,
        )
        .await
        .unwrap();

    loop {
        if sensor.accel_status().await.unwrap().xyz_new_data() {
            let data = sensor.acceleration().await.unwrap();
            let x = data.x_mg();
            let y = data.y_mg();
            let z = data.z_mg();
            if detect_shake(x, y, z) {
                // info!("SHAKE => x:{}, y:{}, z:{}", x, y, z);
                speaker.start_note(Pitch::Named(NamedPitch::A4));
                Timer::after_millis(100).await;
                speaker.stop();
            }
        }
        Timer::after_millis(50).await;
    }
}
```



## Clone the Quick start project
You can clone the quick start project I created and navigate to the project folder and run it.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/bsp-embassy/shake-detect
```

## Flash - `Run Rust Run`
All that's left is to flash the code to your micro:bit and see it in action.

Run the following command from your project folder:
```rust
cargo run
```

Now, try to shake the microbit, it should play a tone when a shake is detected.

## Applications

What else can you do with shake detection? Plenty of fun and creative possibilities!

Here are a few ideas:

- Shake to roll a dice: Each shake generates a random number between 1 and 6.

- Shake to change color or animation: Update an LED pattern or display a new frame each time the micro:bit is shaken.

- Shake to control a game: Use shakes as input for jumping, moving, or triggering actions in simple games.

- Step counter: Detect small, repeated shakes to count steps like a basic pedometer.
