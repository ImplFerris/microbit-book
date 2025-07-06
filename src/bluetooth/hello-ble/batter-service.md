# GATT Service - "service.rs"

Our program defines a single GATT service called the Battery Service.

## Battery Service

The nrf_softdevice crate provides a gatt_service macro along with a characteristic attribute to define GATT services and characteristics. We can specify the UUID for both the service and its characteristics.

In this example, we use the standard 16-bit UUID 0x180F, which is the predefined identifier for the Battery Service . The battery level characteristic uses the standard UUID 0x2A19, which represents the current battery level.

If you're implementing a custom, non-standard service or characteristic, you should generate and use a custom 128-bit UUID instead.

```rust

#[nrf_softdevice::gatt_service(uuid = "180f")]
pub struct BatteryService {
    #[characteristic(uuid = "2a19", read, notify)]
    battery_level: i16,
}
```

We also specify permissions using the read and notify keywords. read allows connected devices to fetch the battery level whenever they want. notify lets us push updates to the client when the battery level changes.

## GATT Server
A GATT server is responsible for hosting services and characteristics that can be accessed by a connected GATT client (e.g., a smartphone or computer).

We define a Server struct with a field of type BatteryService. We then mark this struct with the `#[gatt_server]` attribute macro. This generates all the necessary boilerplate code and provides us with useful functions like Server::new() to initialize the GATT server.

```rust
#[nrf_softdevice::gatt_server]
pub struct Server {
    bas: BatteryService,
}
```

## The full code of service.rs

We've added a method to the Server struct called "notify_battery_value", which will run in the background once a connection is established. This is not the actual battery status. We're just simulating it by increasing and decreasing the battery level to demonstrate.

Inside that function, we call self.bas.battery_level_notify() with the current battery level.  If the client has subscribed to notifications for the battery level characteristic, they'll receive updates automatically. Otherwise, the value is still updated and can be read on demand by the client.

```rust

use defmt::{info, unwrap};
use embassy_time::Timer;
use nrf_softdevice::ble::Connection;

#[nrf_softdevice::gatt_service(uuid = "180f")]
pub struct BatteryService {
    #[characteristic(uuid = "2a19", read, notify)]
    battery_level: i16,
}

#[nrf_softdevice::gatt_server]
pub struct Server {
    bas: BatteryService,
}

impl Server {
    pub async fn notify_battery_value(&self, connection: &Connection) {
        let mut battery_level = 100;
        let mut charging = false;
        loop {
            if battery_level < 20 {
                charging = true;
            } else if battery_level >= 100 {
                charging = false;
            }

            if charging {
                battery_level += 5;
            } else {
                battery_level -= 5;
            }

            match self.bas.battery_level_notify(connection, &battery_level) {
                Ok(_) => info!("Battery Level: {=i16}", &battery_level),
                Err(_) => unwrap!(self.bas.battery_level_set(&battery_level)),
            };

            Timer::after_secs(2).await
        }
    }
}
```
