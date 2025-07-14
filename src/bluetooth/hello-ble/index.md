# Writing Embedded Rust Code to Connect Microbit Bluetooth Low Energy (BLE) with a Phone

We have seen what a BLE stack is and understood the basic concepts. Now it's time to use that knowledge in action. We will write a simple (narrator: the author was lying) project that lets the micro:bit send and receive data from a phone.  

Our program will include one BLE service called `BatteryService`. A connected device (like your phone) can either read the current battery level or subscribe to get updates whenever the battery level changes.

## Create Project from template

We will use Embassy again, but this time without the BSP. Instead, we will work directly with the embassy-nrf HAL.

```sh
cargo generate --git https://github.com/ImplFerris/mb2-template.git --rev 3d07b56
```

- When it prompts for a project name, type something like "hell-ble".

- When it prompts whether to use async, select "true".

- When it prompts you to select between "BSP" or "HAL", select the option "HAL".


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
