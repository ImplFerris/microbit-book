# Implementing the Core logic

We started with a fresh project, and in the last few chapters, we mainly focused on theory. That might have felt a bit dry (or maybe exciting, depending on your perspective). In this section, we will switch gears and focus on writing actual code to keep things hands-on and engaging.

Keep in mind: the code will not compile or run just yet, since we still need to set up a few configurations specific to the microcontroller. But don't worry! We will take it step by step. For now, let's concentrate on building the core logic.

## Imports

Let's begin with the necessary imports. We will use the `DelayNs` and `OutputPin` traits provided by the embedded HAL. The DelayNs trait allows us to add delays between turning the LED on and off, creating the visible blinking effect;without it, the LED would toggle too fast to see. 

As mentioned earlier, the OutputPin trait provides methods(set_low, set_high) to change the state of a microcontroller's output pin. We will use it to toggle the output pin connected to our target LED between low and high states.

```rust
use embedded_hal::{delay::DelayNs, digital::OutputPin};
use microbit::{board::Board, hal::timer::Timer};
```
We will also import the necessary structs from the microbit crate.

## Main function
Now let's update the main function.

First, we acquire access to the micro:bit board peripherals by calling:
```rust
let mut board = Board::take().unwrap();
```
This line gives us a singleton instance of the board, which contains handles to all the microcontroller's hardware peripherals like pins, timers, and more. The take() method returns an Option because the peripherals can only be taken once during the program's lifetime to prevent multiple mutable accesses that could cause conflicts or data races. Calling unwrap() here is safe because we expect to call take() only once in our program.


After obtaining the board peripherals, the next step is to create a timer instance:
```rust
let mut timer = Timer::new(board.TIMER0);
```

### Display Matrix

