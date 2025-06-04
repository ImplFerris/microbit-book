# Embedded HAL

The [`embedded-hal`](https://github.com/rust-embedded/embedded-hal) crate is the heart of the embedded Rust ecosystem. It provides a foundation of common hardware abstraction traits for things like I/O, SPI, I2C, PWM, and timers. These traits create a standard interface that lets high-level drivers, like those for sensors or wireless devices, work across different hardware platforms.

Because drivers are written as generic libraries on top of embedded-hal, they can support a wide range of targets, from Cortex-M and AVR microcontrollers to embedded Linux systems. 

## Example

In the quick-start example, we used embedded-hal traits to control pins and timers on a micro:bit board. The `set_low` and `set_high` functions come from the [OutputPin](https://docs.rs/embedded-hal/1.0.0/embedded_hal/digital/trait.OutputPin.html) trait, and the `delay_ms` function comes from the `DelayNs` trait, both part of embedded-hal.


You might still wonder why not just write `set_low` and `set_high` functions directly without using traits. To illustrate this, consider two versions of a simple function that turns an LED on or off:


```rust
// Example with a concrete pin type (imaginary MicrobitPin)
struct MicrobitPin;

impl MicrobitPin {
    fn set_low(&mut self) {
        // Hardware-specific code to set the pin low
    }
    fn set_high(&mut self) {
        // Hardware-specific code to set the pin high
    }
}
```

Your application/driver code:
```rust
fn control_led_concrete(pin: &mut MicrobitPin, light_up: bool) {
    if light_up {
        pin.set_low();
    } else {
        pin.set_high();
    }
}
```

This is your application code to control the LED. This function only works with the MicrobitPin type. What if you want to port your application or driver to support other microcontrollers? Then you have to write a new crate or add separate logic to handle those. 

```rust
fn control_led_another_mcu(pin: &mut AnotherMcuPin, light_up: bool) {
    if light_up {
        pin.set_low();
    } else {
        pin.set_high();
    }
}
```

Now let's compare it with the embedded-hal trait-based approach:
 
 ```rust
 use embedded_hal::digital::OutputPin;

// This function works with any pin type that implements OutputPin
fn control_led_generic<P: OutputPin>(pin: &mut P, light_up: bool) {
    if light_up {
        let _ = pin.set_low();
    } else {
        let _ = pin.set_high();
    }
}
```

By using the OutputPin trait, this function works on any hardware platform that implements the trait. This makes your code reusable and portable without rewriting it for every board.

This trait-based approach is why embedded-hal is so important in embedded Rust - it provides a common interface that works across different hardware.
