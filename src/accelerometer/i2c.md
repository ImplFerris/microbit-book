# How to Communicate with the LSM303AGR Accelerometer on micro:bit v2 

We learned that the micro:bit v2 has a chip called LSM303AGR for the accelerometer (and magnetometer). But how does the nRF52 processor on the micro:bit actually talk to this chip?

To communicate with the sensor, we'll use a protocol called I2C. Since we haven't covered I2C yet, let's briefly look at what it is. We won't go into too much detail; just enough to understand what we need for this exercise.

## I2C (Inter-Integrated Circuit) Serial Bus

I2C (pronounced "I-squared-C", also written as I²C or IIC) is a simple two-wire communication protocol that allows a microcontroller to exchange data with external devices like sensors, displays, and memory chips.

Think of I2C like a chat system between devices using just two wires.

**The Two Wires:**

- SDA (Serial Data Line) : This is the wire where the actual data travels. It's used to send and receive messages between the nRF52 processor and the sensor.
- SCL (Serial Clock Line) : This is like a traffic light. It tells the devices when it's their turn to speak. Just like cars at an intersection wait for the green light, the devices wait for the clock signal to send or receive data.

<a href="./images/microbit-nrf52-LSM303AGR-i2c.svg"><img alt="microbit LSM303AGR I2C" style="display: block; margin: auto;" src="./images/microbit-nrf52-LSM303AGR-i2c.svg"/></a>

## Who Talks First? Understanding Controller and Device Roles
In I2C communication, one device is in charge of starting and controlling the communication. The others respond when asked. Traditionally, these roles were called master and slave, but there are other alternative terms now used:

- Controller (was "master"): the device that starts communication and controls the timing. In our case, this is the micro:bit's nRF52 chip.

- Peripheral or Device (was "slave"): the chip that listens and responds to the controller. In our case, it is the LSM303AGR sensor.

You can think of the controller as the person asking questions, and the device as the person answering.

## I2C Addresses: How the Controller Knows Who to Talk To
Each device on the I2C bus has a unique address; just like a house on a street has its own number. This is how the controller knows which device it's talking to.

Before sending any data, the controller first sends the device's address. Only the device with that address will respond. All the others stay quiet.

> As stated in the LSM303AGR datasheet, The accelerometer sensor slave address is 0011001b while magnetic sensor slave address is 0011110b. 

Example:
- The LSM303AGR accelerometer's address is 0x19 (in binary: 0011001).
- The magnetometer part has address 0x1E (in binary: 0011110).

So even though both are inside the same chip, the nRF52 can talk to each one separately by using the correct address.

## I2C Bus on Microbit v2 

The micro:bit v2 separates the I2C bus into two parts: internal and external. The internal I2C is used for communication between the nRF52833 processor and onboard components like the motion sensor and interface chip. The external I2C is routed to the edge connector(If you are wondering what is edge connector, refer the [hardware details section](../hardware.md)) for connecting external accessories.

- The internal I2C uses GPIO pin P0.16 on the nRF52833 for SCL (`I2C_INT_SCL`) and P0.08 for SDA (`I2C_INT_SDA`).  
- The external I2C uses GPIO pin P0.26 on the nRF52833 for SCL (`I2C_EXT_SCL`, edge connector pin P19) and P1.00 for SDA (`I2C_EXT_SDA`, edge connector pin P20).

Since the accelerometer is an internal chip, it communicates over the internal I2C bus.

We will be using the "microbit-bsp" crate. In that crate, we can access the internal SDA and SCL lines like this:
```rust
let sda = board.i2c_int_sda;
let scl = board.i2c_int_scl;
```
