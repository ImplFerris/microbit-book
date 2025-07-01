# Built-in Microphone on microbit v2

The micro:bit v2 features an on-board MEMS (Micro-Electro-Mechanical Systems) microphone that allows it to detect sound levels from the surrounding environment. This microphone enables sound-based interactivity, such as reacting to claps, voice, music, or other noise.

A small LED indicator is located on the front of the board, just above the microphone. This LED lights up whenever the microphone is powered on, giving a visual indication that the device is actively listening.

Inside the microbit, the microphone is connected to pin P0.05 of the nRF52833 chip.  In the nRF52833, this pin can work as a normal digital pin or as an analog input (AIN3). The microphone sends its sound signal to this pin, and a part of the chip called the SAADC (Successive Approximation Analog-to-Digital Converter) converts the signal into a number that your program can use.


## Sound Level with BSP

The microbit-bsp crate makes our life easier. It provides a Microphone struct that simplifies working with the built-in microphone. It exposes a method called sound_level(), which enables the microphone and returns the detected sound level.

The returned value ranges from 0 to 255, with higher numbers representing louder sounds. However, this is not a standard unit like decibels. It simply gives a rough idea of how loud the surrounding environment is.


## Embassy Executor and Task Arena Size

The embassy-executor uses a memory area called the [task arena](https://docs.rs/embassy-executor/latest/embassy_executor/#task-arena) to store async tasks. Each time you run a task using async, it takes up space in this arena. If the arena is too small to fit all the tasks and their local variables, the program will crash at runtime.

To avoid this, you need to make sure the arena is large enough to hold all your tasks. When you create a new project from the micro:bit template, it includes the embassy-executor crate with the feature "task-arena-size-1024" by default. This sets the arena size to 1024 bytes (1 KB).

You can increase the arena size by changing the number in that feature. For example:

```toml
features = ["task-arena-size-65536"]
```

We need to increase the task arena size in our program because the "sound_level()" function creates a large buffer to hold audio samples. This buffer uses several kilobytes of memory. 

Here's the part of the function that creates the buffer:
```rust
let mut bufs = [[[0; 1]; 1024]; 2]; // i16
```
This creates an array with 1024 * 2 = 2048 elements. Since each i16 takes 2 bytes, the total size is 2048 * 2 = 4096 bytes (4 KB).

If we keep the default arena size of 1024 bytes, there won't be enough space for this task to run, and the program will crash with a memory error. By increasing the arena size (for example, using task-arena-size-8192), we make sure there's enough memory for the task to run correctly.



