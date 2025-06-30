
## Initialize sensor

Let's initialize the sensor. For this, we first create an I2C interface using the embassy-nrf and BSP crates. Once the I2C interface is instantiated, we pass it to the Lsm303agr driver to initialize the sensor.

Add the necessary imports:
```rust
use embassy_nrf::{self as hal, twim::Twim};
use hal::twim;
use lsm303agr::Lsm303agr;
```

## Two Wire Interface in Master mode (TWIM)
We initialize the I2C-compatible TWIM driver provided by the embassy-nrf crate. This peripheral allows the nRF processor to act as an I2C master and communicate with devices like the LSM303AGR sensor.

We will create a Twim instance, which requires the I2C peripheral (TWISPI0), the interrupt handler (Irqs), the I2C pins (SDA and SCL), and configuration (which we will leave it to default).

```rust
let twim_config = twim::Config::default();
let twim0 = Twim::new(
    board.twispi0,
    Irqs, 
    board.i2c_int_sda, // Internal I2C SDA, GPIO Pin: P0_08
    board.i2c_int_scl, // Internal I2C SCL, GPIO Pin:P0_08
    twim_config,
);
```

## Lsm303agr Driver 

Once the TWIM is initialized, we can pass it to the Lsm303agr::new_with_i2c function, which creates a new instance of the sensor driver using the provided I2C interface.

```rust
let mut sensor = Lsm303agr::new_with_i2c(twim0);
```
