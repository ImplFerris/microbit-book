# Simple Rust Program to Read Values from the Accelerometer

Let's start by writing a simple Rust program that reads and prints values from the accelerometer. We will display the acceleration along the X, Y, and Z axes, expressed in milligravity (mg) units. We will be print the readings to the system console.

In later chapters, we will explore some fun exercises using these values. For now, let's get started!


## Create Project from template

For this exercise, we will be using the microbit-bsp crate with Embassy support. Technically, we could work directly with the embassy-nrf crate, as it is sufficient for this simple example. However, in the next chapter, we will play a sound or display something on the LED matrix. For that, we will rely on the extra features provided by microbit-bsp. So we are layering things now to make those future tasks easier.

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
lsm303agr = { version = "1.1.0", features = ["async"] }
```

The "[lsm303agr](https://docs.rs/lsm303agr/latest/lsm303agr/)" crate is a new dependency we are introducing. It is a platform-agnostic Rust driver for the LSM303AGR sensor, providing a convenient set of functions to read data from the device. We have enabled the "async" feature also for the driver. 

## Update the imports

Before we get started, let's update the imports needed for the I2C peripheral, timing, and the LSM303AGR sensor driver:

```rust
use defmt_rtt as _;
use embassy_executor::Spawner;
use embassy_time::{Delay, Duration, Timer};

// for I2C
use embassy_nrf::{self as hal, twim::Twim};
use hal::twim;

// LSM303AGR Driver
use lsm303agr::{AccelMode, AccelOutputDataRate, Lsm303agr};

// BSP Crate
use microbit_bsp::Microbit;
```

## Initialize the Board

As usual, start by clearing out any existing lines inside the main function that come with the template. Then, add the following line to initialize the board:

```rust
let board = Microbit::default();
```

## I2C Interrupt

We previously discussed [interrupts](../../temperature-sensor/irq.md) in the Temperature Sensor chapter, so by now, you should have a basic understanding of how they work. For I2C communication, we need to enable the interrupt associated with the TWISPI0 peripheral.

>TWISPI0 is a shared hardware peripheral on the nRF52833 that can function as either an I2C (TWI) or SPI interface, depending on how it's configured.

To do this, we use the bind_interrupts! macro from the embassy-nrf crate. This macro defines a unit struct named "Irqs" and connects the TWISPI0 interrupt to the appropriate handler. The macro also implements the required Binding trait for Irqs, which ensures a compile-time type check that the correct interrupt is configured. (If you are not sure what we are talking about here, refer to the [Expanding bind_interrupts! Macro](../../temperature-sensor/irqs-struct.md) section.)

```rust
hal::bind_interrupts!(struct Irqs {
    TWISPI0 => twim::InterruptHandler<hal::peripherals::TWISPI0>;
});
```
