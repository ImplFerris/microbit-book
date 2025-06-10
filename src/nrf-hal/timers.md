# Timers in Embedded Systems

In embedded systems, a **timer** is a special hardware peripheral that counts clock cycles. It helps you measure time or trigger things after a delay. Unlike a simple loop that wastes CPU time, a timer keeps counting in hardware, so the CPU can do other work or even go to sleep.

You can use timers to:

- Blink LEDs at fixed intervals
- Measure how long an operation takes
- Trigger periodic tasks
- Generate precise delays
- Control PWM signals for motors or LEDs

Timers are one of the most useful peripherals in any microcontroller.  Most microcontrollers have multiple hardware timers: TIMER0, TIMER1, etc. Each one works independently.

## Timers on the nRF52833
The nRF52833 microcontroller includes five 32-bit timers with counter mode.  We are not going more in-depth into the inner working of the time for now.

We will use the timer to introduce a delay between turning the LED on and off, to create a blinking effect.

## Modify the code

Now let's update the src/main.rs file to initialize the timer peripheral.

First, add the necessary import:
```rust
use nrf52833_hal::timer::Timer;
```

Inside the main function, add the following line:
```rust
let mut timer0 = Timer::new(peripherals.TIMER0);
```
This line initializes hardware timer TIMER0 using the HAL's Timer abstraction. Later, we will use this instance to create delays between turning the LED on and off.


