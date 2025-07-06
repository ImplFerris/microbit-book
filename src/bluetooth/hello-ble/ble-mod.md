# BLE Module

Let's start by creating the `ble` module.

## ble/mod.rs file

Here, we define a simple function. It calls `softdevice_config` (which we will define shortly in `config.rs`) to get the SoftDevice configuration. Then we enable the SoftDevice using that config. 


```rust
pub mod adv;
pub mod config;

use crate::ble::config::softdevice_config;

use {defmt_rtt as _, panic_probe as _};

use nrf_softdevice::Softdevice;

pub fn get_soft_device() -> &'static mut Softdevice {
    let sd_config = softdevice_config();
    Softdevice::enable(&sd_config)
}

```

This gives us an instance of the `Softdevice` struct.  We will call get_soft_device() from the main function later and do further actions on the instance.


## ble/adv.rs file

Next, we will create a module to prepare the advertisement data.  We already covered the core ideas behind advertising in the [GAP section](../ble-stack/gap.md).

We use `LegacyAdvertisementBuilder` to prepare the advertisement payload. Here, we set the flags to make our device discoverable. For example, we use `GeneralDiscovery` to keep advertising until connected, and `LE_Only` to say we don’t support Bluetooth Classic.

We also add the 16-bit UUID for the `BATTERY` service to let central devices know what we support.  The DEVICE_NAME is the constnat ("implRust") that we will define later in the config.rs file.


```rust
use crate::ble::config::DEVICE_NAME;

use {defmt_rtt as _, panic_probe as _};

use nrf_softdevice::ble::{
    advertisement_builder::{
        Flag, LegacyAdvertisementBuilder, LegacyAdvertisementPayload, ServiceList, ServiceUuid16,
    },
    peripheral,
};

static ADV_DATA: LegacyAdvertisementPayload = LegacyAdvertisementBuilder::new()
    .flags(&[Flag::GeneralDiscovery, Flag::LE_Only])
    .services_16(ServiceList::Complete, &[ServiceUuid16::BATTERY])
    .full_name(DEVICE_NAME)
    .build();

static SCAN_DATA: [u8; 0] = []; 

pub fn get_adv() -> peripheral::ConnectableAdvertisement<'static> {
    peripheral::ConnectableAdvertisement::ScannableUndirected {
        adv_data: &ADV_DATA,
        scan_data: &SCAN_DATA,
    }
}
```

At the end, we return a `ScannableUndirected` advertisement type using `peripheral::ConnectableAdvertisement`. It means our device:

- advertises to any nearby central (not targeting a specific one)  
- accepts connection requests  
- responds to scan requests with extra info
