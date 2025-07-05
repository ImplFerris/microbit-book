# Bluetooth

Bluetooth needs no introduction. You probably use it daily; connecting your wireless headphones with your phone, wireless mouse, smartwatch, and more. In smart homes, Bluetooth helps link different devices. For example, you can control lights, appliances, and thermostats using Bluetooth-enabled apps on your phone.

<div class="alert-box alert-box-info">
    <span class="icon"><i class="fa fa-info"></i></span>
    <div class="alert-content">
        <b class="alert-title">Did you know?</b>
            <p>Bluetooth was named after King Harald "Bluetooth" Gormsson. During a meeting, Jim Kardach from Intel suggested it as a temporary code name. Kardach later said, "King Harald Bluetooth…was famous for uniting Scandinavia just as we intended to unite the PC and cellular industries with a short-range wireless link"</p>
    </div>
</div>


## Categories

Bluetooth technology is divided into two main types: Bluetooth Classic and Bluetooth Low Energy (BLE).

### Bluetooth Classic
Bluetooth Classic is the original version of Bluetooth, commonly used in devices that require continuous data transfer, such as wireless headsets, speakers, and mouse. Before BLE was introduced, it was simply called "Bluetooth," but now it's referred to as Bluetooth Classic to distinguish it from BLE. It offers higher data rates, making it ideal for real-time applications like audio streaming.

### Bluetooth Low Energy (BLE)
BLE is designed for low power consumption, making it great for devices that send small amounts of data from time to time. This is especially useful for battery-powered IoT devices, such as fitness trackers and environmental sensors. Compared to Classic, BLE has lower latency, meaning it takes less time to start sending or receiving data after a connection is made.

### Dual Mode
Many modern devices support both Bluetooth Classic and BLE, a feature known as "Dual Mode." For example, a smartphone might use Classic for streaming music and BLE to connect to a smartwatch.

In general, Bluetooth Classic is better suited for continuous data transmission, such as real-time audio and video, while BLE is ideal for low-power communication with health monitors, sensors, and other small gadgets.

### Microbit Bluetooth

The micro:bit supports Bluetooth 5.1 using Bluetooth Low Energy (BLE) via the [Nordic S113 SoftDevice](https://www.nordicsemi.com/Products/Development-software/s113). While it doesn't support Bluetooth Classic, its BLE capabilities are enough for many practical applications, such as sending sensor data to a phone, controlling LEDs from an app, or communicating with other micro:bits.

> ⚠️ Heads up: This might be one of the more challenging chapters in this book. Working with Bluetooth Low Energy (BLE) is not simple. There are many things to learn and understand. Before we can run our Rust program, we need to flash the SoftDevice (a special Bluetooth firmware) onto the micro:bit. We also need to update the [memory.x](/intro-walkthrough/are-we-there-yet.html#memoryx) file to make sure our program doesn't overwrite the space used by the SoftDevice.
