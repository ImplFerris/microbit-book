
# GPIO Pins

To turn on the first LED, we had to set the first column to LOW and then set the first row to HIGH. This would complete the circuit and light up the LED.

When we were using the BSP crate, it was pretty straightforward. The BSP gave us names like row1, row2 or col1, col2 so we could easily use them in code. We could also use a 2D array to represent the whole LED matrix.  

But now when using the HAL, we work directly with GPIO pins. Instead of row1 or col1, we use pin names like p0_28, p0_21, etc. These are the actual pins on the microcontroller that we configure as inputs or outputs.

## What is GPIO?

GPIO stands for General Purpose Input Output. These are pins on a microcontroller that we can control from our code.

Each GPIO pin can be:

- An output : We can set it to HIGH (like power) or LOW (like ground)
- An input : We can read if it is HIGH or LOW from a sensor or button

Think of GPIO pins like switches that we control from our code. When we set a pin to HIGH, it acts like a power source. When we set it to LOW, it acts like a path to ground.

By setting some pins HIGH and others LOW, we can control LEDs, buttons, sensors, and more.

## How to Know Which GPIO to Use?

To figure out which GPIO pins to use, you usually check the datasheet or technical reference manual for the microcontroller or board you are working with.  You can also refer the [Pinmap](https://tech.microbit.org/hardware/schematic/#v2-pinmap) in the micro:bit documentation.

To make things easier, here is a table that shows how each row and column in the LED matrix connects to the actual GPIO pins on the nRF52833 microcontroller.


| Matrix Role | Role   | Port | Pin  |
|-------------|--------|------|------|
| ROW1        | Source | p0   | 21   |
| ROW2        | Source | p0   | 22   |
| ROW3        | Source | p0   | 15   |
| ROW4        | Source | p0   | 24   |
| ROW5        | Source | p0   | 19   |
| COL1        | Sink   | p0   | 28   |
| COL2        | Sink   | p0   | 11   |
| COL3        | Sink   | p0   | 31   |
| COL4        | Sink   | p1   | 05   |
| COL5        | Sink   | p0   | 30   |

The Pin column refers to the GPIO pin number. Port is the term we have not introduced yet. It is just a group of GPIO pins that are managed together by the microcontroller. Microcontrollers often organize their GPIO pins into ports , such as Port 0 (p0) or Port 1 (p1) . Each port can control multiple pins.

Our goal is to turn on the first LED, which is in the first row and first column. For the first row, the GPIO pin is p0_21 . For the first column, the GPIO pin is p0_28 .

## Modify the Code

Now, let's modify the `src/main.rs` file to initialize the GPIO port and the specific pins we need.

First, add the necessary import:
```rust
use nrf52833_hal::gpio::Level; 
use nrf52833_hal as hal;
```

Here, `Level` is an enum that represents the logic level of a pin: either Level::High (for a logical high voltage) or Level::Low (for a logical low voltage). 

We also import the `nrf52833_hal` crate using the alias `hal`. This makes it easier to refer to its modules and types throughout the code without repeatedly writing the full crate name.

Inside the `main` function, add the following code:
```rust
let port0 = hal::gpio::p0::Parts::new(peripherals.P0);
let _col1 = port0.p0_28.into_push_pull_output(Level::Low);
let mut row1 = port0.p0_21.into_push_pull_output(Level::High);
```

Here's what this does:

- `port0` gives us access to the individual pins in Port 0.
- `_col1` is set as a **push-pull output** and driven **LOW**, which means the pin is connected to ground. This "activates" the column by allowing current to flow into it.
- `row1` is also set as a **push-pull output**, but driven **HIGH**, which means the pin is connected to the power supply (like 3.3V). This "selects" the row by providing the source of current.


### What is a Push-Pull Output?

A **push-pull output** is a common electrical configuration for GPIO pins on microcontrollers. In this mode, the pin can actively drive the output both high and low:

- When set **high**, the pin is connected to the supply voltage (typically 3.3V or 5V), allowing it to source current to connected components.
- When set **low**, the pin is connected to ground, allowing it to sink current.

This means the pin is never left floating and always has a definite voltage level. It is ideal for driving digital outputs like LEDs, relays, or for controlling logic signals.

Unlike **open-drain** or **open-collector** outputs, which can only pull the line low and require an external pull-up resistor to go high, **push-pull** does both, making it simple and reliable for general-purpose output.
