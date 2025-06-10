# Peripherals

In embedded systems, peripherals are hardware components that extend the capabilities of a microcontroller (MCU). They allow the MCU to interact with the outside world by handling inputs and outputs, communication, timing, and more.

While the CPU is responsible for executing program logic, peripherals do the heavy lifting of interacting with hardware, often offloading work from the CPU. This allows the CPU to focus on critical tasks while peripherals handle specialized functions independently or with minimal supervision.

## Offloading

Offloading refers to the practice of delegating certain tasks to hardware peripherals instead of doing them directly in software via the CPU. This improves performance, reduces power consumption, and enables concurrent operations. For example:

- A UART peripheral can send and receive data in the background using DMA (Direct Memory Access), while the CPU continues processing other logic.
- A Timer can be configured to generate precise delays or periodic interrupts without CPU intervention.
- A PWM controller can drive a motor continuously without the CPU constantly toggling pins.

Offloading is a key design strategy in embedded systems to make efficient use of limited processing power.

## Common Types of Peripherals

Here are some of the most common types of peripherals found in embedded systems:

| Peripheral | Description |
|----------|-------------|
| **GPIO** (General Purpose Input/Output) | Digital pins that can be configured as inputs or outputs to interact with external hardware like buttons, LEDs, and sensors. |
| **UART** (Universal Asynchronous Receiver/Transmitter) | Serial communication interface used for sending and receiving data between devices, often used for debugging and connecting modules like Bluetooth. |
| **SPI** (Serial Peripheral Interface) | High-speed synchronous communication protocol used to connect microcontrollers to peripherals like SD cards, displays, and sensors using a master-slave architecture. |
| **I2C** (Inter-Integrated Circuit) | Two-wire serial communication protocol used for connecting low-speed peripherals such as sensors and memory chips to a microcontroller. |
| **ADC** (Analog-to-Digital Converter) | Converts analog signals from sensors or other sources into digital values that the microcontroller can process. |
| **PWM** (Pulse Width Modulation) | Generates signals that can control power delivery, used commonly for LED dimming, motor speed control, and servo actuation. |
| **Timer** | Used for generating delays, measuring time intervals, counting events, or triggering actions at specific times. |
| **RTC** (Real-Time Clock) | Keeps track of current time and date even when the system is powered off, typically backed by a battery. |


## Peripherals in Rust

In embedded Rust, peripherals are accessed using a singleton model. One of Rust's core goals is safety, and that extends to how it manages hardware access. To ensure that no two parts of a program can accidentally control the same peripheral at the same time, Rust enforces exclusive ownership through this singleton approach.

### The Singleton Pattern
The singleton pattern ensures that only one instance of each peripheral exists in the entire program. This avoids common bugs caused by multiple pieces of code trying to modify the same hardware resource simultaneously.

We have already encountered this pattern in the BSP crate, where the entire board (including all peripherals, pins, and configuration) is wrapped as a singleton.

In the nRF hal, the peripherals are also exposed as a singleton object:


## Modify the Code

Let's now modify the `src/main.rs` file to create an instance of the microcontroller peripherals using Rust's singleton model.

### Step 1: Import the Peripherals

First, open `src/main.rs` and add the necessary import:

```rust
use nrf52833_hal::pac::Peripherals;
```

### Step 2: Take Ownership of the Peripherals
Inside the main function, add the following line:

```rust
let peripherals = Peripherals::take().unwrap();
```

This line does the following:

- Takes ownership of the device's peripherals.
- Returns an instance of the `Peripherals` struct, which contains access to all hardware blocks like GPIO and others.
- Can only be called once during the program's lifetime. If called again, it will return `None`.

