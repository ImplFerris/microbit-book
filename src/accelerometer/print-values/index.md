# Simple Rust Program to Read Values from the Accelerometer

Let's start by writing a simple Rust program that reads and prints values from the accelerometer. We will display the acceleration along the X, Y, and Z axes, expressed in milligravity (mg) units. We will be print the readings to the system console.

In later chapters, we will explore some fun exercises using these values. For now, let's get started!

## Create Project from template

For this project, we will be using `microbit-bsp` (with Embassy). To generate a new project using the template, run the following command:

```sh
cargo generate --git https://github.com/ImplFerris/mb2-template.git --rev 3d07b56
```

- When it prompts for a project name, type something like "accelerometer-print".

- When it prompts whether to use async, select "true".

- When it prompts you to select between "BSP" or "HAL", select the option "BSP".

## Update Cargo.toml

The "[lsm303agr](https://docs.rs/lsm303agr/latest/lsm303agr/)" crate is a new dependency we are introducing. It is a platform-agnostic Rust driver for the LSM303AGR sensor, providing a convenient set of functions to read data from the device. We have enabled the "async" feature also for the driver. 

```toml
lsm303agr = { version = "1.1.0", features = ["async"] }
static_cell = { version = "2" }
```

We also add the static_cell crate, which provides tools for safely creating and accessing static memory. We use it to allocate a fixed-size RAM buffer required by the TWIM (I2C) driver. This buffer must have a 'static lifetime and live in RAM, which static_cell helps manage safely.

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
