# Dependencies


## nrf-softdevice crate
This crate provides Rust bindings for Nordic's closed-source SoftDevice Bluetooth stack. The SoftDevice is a precompiled C binary that runs first on startup and then hands control to your app. It's battle tested, pre qualified for bluetooth certification.  Learn more: [nrf-softdevice GitHub](https://github.com/embassy-rs/nrf-softdevice/).

We will use this crate to handle Bluetooth in our project. To use it, we must specify the following features for the crate:

- Exactly one SoftDevice model. Depending on the model, the crate supports different roles. The micro:bit uses the s113 SoftDevice, so we will have to use that. For this model, the crate supports only the peripheral role. It does not support central mode.

- Exactly one supported nRF chip. For the micro:bit, we already know it uses the nrf52833.

So, we will have to update the Cargo.toml with the following dependencies:

```toml
# nrf-softdevice = { version = "0.1.0", features = ["ble-peripheral", "ble-gatt-server", "s113", "nrf52833", "critical-section-impl", "defmt"] }
# To fix a conflict issue, using this latest (at the moment) revision
nrf-softdevice = { git = "https://github.com/embassy-rs/nrf-softdevice/", rev = "5949a5b", features = [
    "ble-peripheral",
    "ble-gatt-server",
    "s113",
    "nrf52833",
    "defmt",
] }
nrf-softdevice-s113 = { version = "0.1.2" }
```

Along with that, we've enabled a few additional features like ble-peripheral and ble-gatt-server which are needed for our use case.

## StaticCell crate
The "StaticCell" crate is useful when you need to initialize a variable at runtime but require it to have a static lifetime.

Update the Cargo.toml with the following dependencies:

```toml
static_cell = "2"
```

## Heapless crate

The main idea behind the heapless crate is that it uses data structures that don't need dynamic memory (no heap). Instead, everything is stored in a fixed-size memory area.

Update the Cargo.toml with the following dependencies:
```toml
heapless = "0.8"
```

For example, heapless::Vec is like Rust's regular Vec, but with one big difference: it has a fixed maximum size that you decide ahead of time, and it can't grow beyond that. This is especially useful in Embedded Rust, where heap memory is often not available

## Futures

The futures crate provides core tools for writing asynchronous code. We use "select!" and "pin_mut!" macros from it to run the battery level notifier and GATT server tasks concurrently.

```toml
futures = { version = "0.3.29", default-features = false }
```

