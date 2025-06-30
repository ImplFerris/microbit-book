## Printing Accelerometer values

We will call the "init" function on the sensor instance. This will initialize the register and prepare the accelerometer for the reading.

```rust
sensor.init().await.unwrap();
```

## Configuring Operating Mode and Output Data Rate (ODR)

Next, we will configure the accelerometer's operating mode and output data rate (ODR).

As described in the datasheet (page 27), the LSM303AGR supports three accelerometer operating modes: high-resolution mode, normal mode, and low-power mode.

**High-resolution mode** provides the finest measurement precision with a resolution of 12 bits. This mode is highly sensitive to small changes in motion or tilt, making it suitable for applications where accuracy is important. However, this increased precision comes with higher power consumption compared to the other modes. 

**The output data rate (ODR)** determines how frequently the sensor provides new acceleration data. Setting the ODR to 50 Hz means the sensor updates its readings 50 times per second. This rate is responsive enough for most human-scale activities like walking, tilting, or detecting gestures, while keeping power consumption relatively low.

```rust
 sensor
    .set_accel_mode_and_odr(
        &mut Delay,
        AccelMode::HighResolution,
        AccelOutputDataRate::Hz50,
    )
    .await
    .unwrap();
```
At a 50 Hz output data rate, the turn-on time is approximately 7 ms, and the current consumption in high-resolution mode is about 12.6 µA. Choosing high-resolution mode with a 50 Hz ODR offers a well-balanced trade-off between accuracy and energy efficiency. It allows the system to capture detailed motion information without overwhelming the processor or draining the battery quickly.

## Read values

Inside a loop, we will check if a new accelerometer reading is available using the snippet `sensor.accel_status().await.unwrap().xyz_new_data()`.
The "accel_status" function reads the status register of the accelerometer, and the "xyz_new_data" function checks a specific flag within that register to determine whether new data is available for all three axes: X, Y, and Z.

If new data is available, we use the "acceleration" function to get the latest X, Y, and Z values. These values are then converted from raw sensor data into milligravity (mg). 

## The full code

Here is the complete code that puts everything together:

```rust
#![no_std]
#![no_main]

use embassy_nrf::{self as hal, twim::Twim};
use hal::twim;

use defmt_rtt as _;
use embassy_executor::Spawner;
use embassy_time::{Delay, Duration, Timer};
use lsm303agr::{AccelMode, AccelOutputDataRate, Lsm303agr};
use microbit_bsp::Microbit;

#[panic_handler]
fn panic(panic_info: &core::panic::PanicInfo) -> ! {
    defmt::info!("{:?}", panic_info);
    loop {}
}

hal::bind_interrupts!(struct Irqs {
    TWISPI0 => twim::InterruptHandler<hal::peripherals::TWISPI0>;
});

#[embassy_executor::main]
async fn main(_spawner: Spawner) -> ! {
    let board = Microbit::default();

    let twim_config = twim::Config::default();
    let twim0 = Twim::new(
        board.twispi0,
        Irqs,
        board.i2c_int_sda,
        board.i2c_int_scl,
        twim_config,
    );

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
            // milli-g values
            let x = data.x_mg();
            let y = data.y_mg();
            let z = data.z_mg();
            defmt::info!("x:{}, y:{}, z:{}", x, y, z);
        }
        Timer::after(Duration::from_secs(1)).await;
    }
}
```

## Clone the existing project
You can also clone (or refer) project I created and navigate to the `bsp-embassy/accelerometer-print` folder.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/bsp-embassy/accelerometer-print
```


## Run

You can flash the program into the micro:bit and should see the readings getting printed on your computer

```sh
cargo run
```

Try changing the orientation of the board, shaking it, or tilting it in different directions. You will see the X, Y, and Z values change based on how the board moves.
