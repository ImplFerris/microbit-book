# Built-in Microphone on microbit v2

The micro:bit v2 features an on-board MEMS (Micro-Electro-Mechanical Systems) microphone that allows it to detect sound levels from the surrounding environment. This microphone enables sound-based interactivity, such as reacting to claps, voice, music, or other noise.

A small LED indicator is located on the front of the board, just above the microphone. This LED lights up whenever the microphone is powered on, giving a visual indication that the device is actively listening.

## Pin and ADC 

Inside the microbit, the microphone is connected to pin P0.05 of the nRF52833 chip.  In the nRF52833, this pin can work as a normal digital pin or as an analog input (AIN3). 

The microphone sends its sound signal to this pin, and a part of the chip called the SAADC (Successive Approximation Analog-to-Digital Converter) converts the signal into a number that your program can use. We will explore ADCs (Analog to Digital Converters) in more detail in later chapters, but for now, just know that the ADC allows us to read sound levels as numeric values.

## Sound Level with BSP

The microbit-bsp crate makes our life easier. It provides a Microphone struct that simplifies working with the built-in microphone. It exposes a method called sound_level(), which enables the microphone and returns the detected sound level.

The returned value ranges from 0 to 255, with higher numbers representing louder sounds. However, this is not a standard unit like decibels. It simply gives a rough idea of how loud the surrounding environment is.

