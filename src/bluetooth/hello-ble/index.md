# Writing Embedded Rust Code to Connect Microbit Bluetooth Low Energy (BLE) with a Phone

We have seen what a BLE stack is and understood the basic concepts. Now it's time to use that knowledge in action. We will write a simple (narrator: the author was lying) project that lets the micro:bit send and receive data from a phone.  

Our program will include one BLE service called `BatteryService`. A connected device (like your phone) can either read the current battery level or subscribe to get updates whenever the battery level changes.

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

## Project Structure

This exercise is adapted from the `ble_bas_peripheral_notify.rs` example in the `nrf-softdevice` repository, with some restructuring for clarity. You can explore more examples [here](https://github.com/embassy-rs/nrf-softdevice/tree/master/examples).

Below is the structure of the `src` folder we will build. We'll create a `ble` module to hold common Bluetooth-related setup and configuration, and a separate `service.rs` file for the battery service logic.

```sh
.
├── ble
│   ├── adv.rs
│   ├── config.rs
│   └── mod.rs
├── main.rs
└── service.rs

1 directory, 5 files
```
