# Temperature Sensor

A temperature sensor is an input device used to measure temperature. You can find them in many places around your home, such as in thermostats that control heating and cooling systems. Many temperature sensors are built into devices with a display, so you can directly see the temperature reading.

The micro:bit includes a temperature sensor inside its nRF52 processor. It measures the chip's internal temperature, which gives an approximation of the surrounding air temperature.

> Note: The reading reflects the internal temperature of the chip; It is not a direct measurement of the ambient air temperature. However, it gives us an approximate idea of the surrounding temperature. 

The sensor has an accuracy of around +/-5°C (uncalibrated) and can sense temperatures in the range of -40°C to 105°C.

## Create Project from template

We will use Embassy again, but this time without the BSP. Instead, we will work directly with the embassy-nrf HAL.

```sh
cargo generate --git https://github.com/ImplFerris/mb2-template.git --rev 3d07b56
```

- When it prompts for a project name, type something like "temperature".

- When it prompts whether to use async, select "true".

- When it prompts you to select between "BSP" or "HAL", select the option "HAL".


## The Full code

This time, we will jump straight into the full code example. The reason is simple: the concepts involved here would take a fair amount of theory to explain first. So instead, we will start by running the code, and then we will break it down step by step.

In this example, we will use the "TEMP" peripheral (the temperature sensor) exposed by the embassy-nrf HAL. The HAL also provides a struct called "Temp" which allows us to interact with the temperature hardware. 

But there's one more thing: we also need to set up something called an Interrupt Request Handler. Yes, this is a new concept we haven't discussed yet. If you have some experience with interrupt handlers, great. If not, don't worry; we will go over what it means and how it works in detail.

```rust
#![no_std]
#![no_main]

use defmt::info;
use embassy_executor::Spawner;
use embassy_time::Timer;
use {defmt_rtt as _, panic_probe as _};

use embassy_nrf::{
    bind_interrupts,
    temp::{self, Temp},
};

bind_interrupts!(struct Irqs {
    TEMP => temp::InterruptHandler;
});

#[embassy_executor::main]
async fn main(_spawner: Spawner) -> ! {
    let p = embassy_nrf::init(Default::default());
    let mut temp = Temp::new(p.TEMP, Irqs);

    loop {
        let value = temp.read().await;
        info!("temperature: {}℃", value.to_num::<u16>());
        Timer::after_secs(1).await;
    }
}
```

In the loop, we just read the temperature and print it using the "info!" macro from defmt. Once you flash this onto the micro:bit, it'll start printing temperature readings to your computer's console. Pretty cool, right? This is something we haven't really taken advantage of before. But yes, you can absolutely run a program on your board and see live logs on your computer!


## Clone the existing project
You can also clone (or refer) project I created and navigate to the `hal-embassy/temperature` folder.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/hal-embassy/temperature
```


## Run

You can flash the program into the micro:bit and should see the logs getting printed on your computer

```sh
cargo run
```

The temperature you see here won't be the exact room temperature. Like we mentioned earlier, this sensor measures the internal temperature of the chip, not the surrounding air. So if you want to get a more accurate room temperature, you'll need to calibrate it yourself. That means testing it in different environments, comparing it with a real thermometer, and then adjusting the value in your code based on that. 

We can also buy and use external sensors, if we want more accurate temperature readings. We'll explore how to use them in the future chapters.

