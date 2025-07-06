# Softdevice Config - ble/config.rs

Now let's set up the configuration needed to run the SoftDevice. All of the following code goes inside the `ble/config.rs` file.

## Imports

```rust
use {defmt_rtt as _, panic_probe as _};
use core::mem;
use nrf_softdevice::raw;
```

## Device Name

First, we define the device name. This will show up when other devices scan for our micro:bit. You are free to modify it with any valid name. 

```rust
pub const DEVICE_NAME: &str = "implRust";
```

## Clock Config

Bluetooth requires precise timing to manage things like advertising, scanning, and maintaining a connection. That timing depends on a low-frequency clock. Here, we configure the internal RC oscillator as the clock source.

```rust
const fn clock_config() -> Option<raw::nrf_clock_lf_cfg_t> {
    Some(raw::nrf_clock_lf_cfg_t {
        source: raw::NRF_CLOCK_LF_SRC_RC as u8,
        // rc_ctiv: 16,
        rc_ctiv: 4,
        rc_temp_ctiv: 2,
        // accuracy: raw::NRF_CLOCK_LF_ACCURACY_500_PPM as u8,
        accuracy: raw::NRF_CLOCK_LF_ACCURACY_20_PPM as u8,
    })
}
```
The internal RC clock isn't very precise and can slowly become inaccurate over time or with temperature changes (drift over time). To fix that, we tell it to regularly recalibrate:

- rc_ctiv controls how often the chip performs periodic recalibration to correct any timing drift.

- rc_temp_ctiv adds another trigger based on temperature changes, so the chip recalibrates if it detects noticeable thermal variation.

- accuracy: 20 PPM tells the BLE stack how precise the clock is expected to be. A lower PPM value means better timing accuracy, which helps Bluetooth work more reliably.


## GAP Connection Config

This sets how many Bluetooth connections your device can handle and how much time is allocated to each connection.

```rust
const fn gap_conn_config() -> Option<raw::ble_gap_conn_cfg_t> {
    Some(raw::ble_gap_conn_cfg_t {
        conn_count: 2,
        event_length: 24,
    })
}
```

- conn_count: 2 means the device can maintain up to 2 simultaneous Bluetooth connections.

- event_length sets how long (in 1.25 millisecond units ) the chip reserves for handling each connection during every Bluetooth communication cycle. A value of 24 gives about 30 milliseconds per interval, which is enough time to send and receive data reliably without dropping the connection.


## GAP Device Name Setup
This config sets the Bluetooth device name that other devices (like your phone) will see when scanning. By default, SoftDevice uses a generic name, but here we override it with our custom name.

```rust
fn gap_device_name() -> Option<raw::ble_gap_cfg_device_name_t> {
    Some(raw::ble_gap_cfg_device_name_t {
        p_value: DEVICE_NAME.as_ptr() as _,
        current_len: DEVICE_NAME.len() as u16,
        max_len: DEVICE_NAME.len() as u16,
        write_perm: unsafe { mem::zeroed() },
        _bitfield_1: raw::ble_gap_cfg_device_name_t::new_bitfield_1(
            raw::BLE_GATTS_VLOC_STACK as u8,
        ),
    })
}
```
- p_value points to the memory where our custom device name ("implRust") is stored.
- current_len and max_len define the current and maximum length of the name.
- write_perm controls whether the name can be changed by connected clients. We're using zeroed() here to disable write access. 
- _bitfield_1 includes vloc, which sets where the name is stored. We use BLE_GATTS_VLOC_STACK, meaning the name is stored in flash (non-volatile memory) and not writable at runtime.
 

## GAP Role Count

This configuration sets limits on how many Bluetooth operations your device can handle at the same time.
```rust
const fn gap_role_count() -> Option<raw::ble_gap_cfg_role_count_t> {
    Some(raw::ble_gap_cfg_role_count_t {
        adv_set_count: raw::BLE_GAP_ADV_SET_COUNT_DEFAULT as u8,
        periph_role_count: raw::BLE_GAP_ROLE_COUNT_PERIPH_DEFAULT as u8,
    })
}
```

- The adv_set_count is how many advertising handles we want. We leave it to default. 

- periph_role_count defines how many devices the micro:bit can be connected to at the same time while acting as a peripheral. Again, we use the default setting (which allows one connection).


## GATT Connection Config
This sets the size of the ATT (Attribute Protocol) packet. In simple terms, it controls how much data can be sent or received in one go over BLE.

```rust
const fn gatt_conn_config() -> Option<raw::ble_gatt_conn_cfg_t> {
    Some(raw::ble_gatt_conn_cfg_t { att_mtu: 256 })
}
```
att_mtu stands for Attribute Maximum Transmission Unit . It defines how much data can be sent in one go between devices. The default MTU is 23 bytes, which is quite small. We're bumping it up to 256 so that we can send or receive larger chunks of data in a single BLE packet. This helps improve throughput and reduces the overhead of sending tiny pieces

## GATT Attribute Table Size
This configuration sets the size of the attribute table , which is used to store data like service definitions, characteristics, and descriptors that other devices can read or write over BLE.

```rust
const fn gatts_attr_tab_size() -> Option<raw::ble_gatts_cfg_attr_tab_size_t> {
    Some(raw::ble_gatts_cfg_attr_tab_size_t {
        attr_tab_size: raw::BLE_GATTS_ATTR_TAB_SIZE_DEFAULT,
    })
}
```
We're using the default size, which is usually enough for most small BLE setups. But if you add many services, you may need to increase this. Make sure the size is a multiple of 4, or it will throw an error.

## Putting It All Together

This function gathers all the configs we defined above and builds the final config that will be passed to Softdevice::enable()

```rust
// Softdevice config
pub fn softdevice_config() -> nrf_softdevice::Config {
    nrf_softdevice::Config {
        clock: clock_config(),
        conn_gap: gap_conn_config(),
        conn_gatt: gatt_conn_config(),
        gatts_attr_tab_size: gatts_attr_tab_size(),
        gap_role_count: gap_role_count(),
        gap_device_name: gap_device_name(),
        ..Default::default()
    }
}
```


## The full content of the config.rs

```rust
use {defmt_rtt as _, panic_probe as _};

use core::mem;

use nrf_softdevice::raw;

pub const DEVICE_NAME: &str = "implRust";

const fn clock_config() -> Option<raw::nrf_clock_lf_cfg_t> {
    Some(raw::nrf_clock_lf_cfg_t {
        source: raw::NRF_CLOCK_LF_SRC_RC as u8,
        // rc_ctiv: 16,
        rc_ctiv: 4,
        rc_temp_ctiv: 2,
        // accuracy: raw::NRF_CLOCK_LF_ACCURACY_500_PPM as u8,
        accuracy: raw::NRF_CLOCK_LF_ACCURACY_20_PPM as u8,
    })
}

const fn gap_conn_config() -> Option<raw::ble_gap_conn_cfg_t> {
    Some(raw::ble_gap_conn_cfg_t {
        conn_count: 2,
        event_length: 24,
    })
}

const fn gatt_conn_config() -> Option<raw::ble_gatt_conn_cfg_t> {
    Some(raw::ble_gatt_conn_cfg_t { att_mtu: 256 })
}

fn gap_device_name() -> Option<raw::ble_gap_cfg_device_name_t> {
    Some(raw::ble_gap_cfg_device_name_t {
        p_value: DEVICE_NAME.as_ptr() as _,
        current_len: DEVICE_NAME.len() as u16,
        max_len: DEVICE_NAME.len() as u16,
        write_perm: unsafe { mem::zeroed() },
        _bitfield_1: raw::ble_gap_cfg_device_name_t::new_bitfield_1(
            raw::BLE_GATTS_VLOC_STACK as u8,
        ),
    })
}

const fn gap_role_count() -> Option<raw::ble_gap_cfg_role_count_t> {
    Some(raw::ble_gap_cfg_role_count_t {
        adv_set_count: raw::BLE_GAP_ADV_SET_COUNT_DEFAULT as u8,
        periph_role_count: raw::BLE_GAP_ROLE_COUNT_PERIPH_DEFAULT as u8,
    })
}

const fn gatts_attr_tab_size() -> Option<raw::ble_gatts_cfg_attr_tab_size_t> {
    Some(raw::ble_gatts_cfg_attr_tab_size_t {
        attr_tab_size: raw::BLE_GATTS_ATTR_TAB_SIZE_DEFAULT,
    })
}

// Softdevice config
pub fn softdevice_config() -> nrf_softdevice::Config {
    nrf_softdevice::Config {
        clock: clock_config(),
        conn_gap: gap_conn_config(),
        conn_gatt: gatt_conn_config(),
        gatts_attr_tab_size: gatts_attr_tab_size(),
        gap_role_count: gap_role_count(),
        gap_device_name: gap_device_name(),
        ..Default::default()
    }
}

```

## Reference

- [One minute to understand BLE MTU data package](https://devzone.nordicsemi.com/nordic/nordic-blog/b/blog/posts/one-minute-to-understand-ble-mtu-data-package)
