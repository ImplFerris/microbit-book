# Main

Now, we will work on the main.rs file. First, We modify the main.rs file with the following required imports:

```rust
#![no_std]
#![no_main]
mod ble;
mod service;
use crate::ble::get_soft_device;
use crate::service::*;

use {defmt_rtt as _, panic_probe as _};

use defmt::{info, unwrap};
use embassy_executor::Spawner;
use embassy_nrf::interrupt;
use futures::future::{Either, select};
use futures::pin_mut;
use nrf_softdevice::Softdevice;
use nrf_softdevice::ble::{gatt_server, peripheral};
```

## Priority

To ensure safe interaction between the SoftDevice (which handles BLE) and your application code, the application must run at a lower interrupt priority than the SoftDevice .

The SoftDevice uses high interrupt priority levels for BLE operation. The application must not use a higher or equal priority level for its own interrupts.

The following function configures the nRF driver to use priority level 2 for key hardware interrupts like GPIOTE and TIMER:
```rust
// Application must run at a lower priority than softdevice
fn nrf_config() -> embassy_nrf::config::Config {
    let mut config = embassy_nrf::config::Config::default();
    config.gpiote_interrupt_priority = interrupt::Priority::P2;
    config.time_interrupt_priority = interrupt::Priority::P2;
    config
}
```

Clear any existing code inside the main function and add this line to initialize the HAL with this config:

```rust
async fn main(spawner: Spawner) {
    let _ = embassy_nrf::init(nrf_config());

    // We will add remaining code here followed by the above line
}
```

## Initialize Softdevice

We first get the Softdevice instance by calling get_soft_device function, which we defined in the ble module. We use this to initialize the Server struct, which we defined in the service module.

Then we run sd.run() in the background with the help of embassy task.
 
```rust
let sd = get_soft_device();
let server = unwrap!(Server::new(sd));
unwrap!(spawner.spawn(softdevice_task(sd)));
```

We define the task like this:
```rust
#[embassy_executor::task]
pub async fn softdevice_task(sd: &'static Softdevice) -> ! {
    sd.run().await
}
```
