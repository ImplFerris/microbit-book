# Generic Access Profile (GAP)

GAP (Generic Access Profile) is a set of rules that control how Bluetooth Low Energy (BLE) devices discover, connect, and communicate with each other.

## BLE Communication Types

BLE supports two main ways to communicate: **connected communication** and **broadcast communication**.

**Connected Communication :** Two devices form a direct connection, allowing them to send and receive data both ways. For example, a smartwatch connects to a phone and continuously shares data like heart rate, notifications, and step count.

**Broadcast Communication:** A device sends data to all nearby devices without making a direct connection. For example, a Bluetooth beacon in a store broadcasts promotional messages to all phones in range.


## Device Roles

Imagine these roles like in real-world human communication. Just as people interact in different ways depending on their roles in a conversation, Bluetooth Low Energy (BLE) devices have specific roles.

**📢 Broadcaster (connection-less)**: Sends out information (advertisements) but cannot be connected to.  
For example, a beacon in a shopping mall continuously sends discount offers to nearby smartphones. The phones can receive the offers but cannot connect to the beacon.  

**📡 Observer (connection-less)**: Listens for Bluetooth advertisements but cannot connect to other devices.  
For example, a smartphone app scans for beacons to detect nearby stores but does not connect to them.  

**📱 Central (connection-oriented)**: This device searches for other devices, connects to them, or reads their advertisement data. It usually has more processing power and resources. It can handle multiple connections at the same time.  

For example, a smartphone connects to a smartwatch, a fitness tracker, and wireless earbuds simultaneously.  

**⌚ Peripheral (connection-oriented)**: This device broadcasts advertisements and accepts connection requests from central devices. 
For example, a fitness tracker advertises itself so a smartphone can find and connect to it for syncing health data.  

<img style="display: block; margin: auto;" alt="Central And Peripherals" src="./images/ble-central-peripheral.jpg"/>


## BLE Peripheral Discovery Modes & Advertisement Flags

A BLE peripheral can be in different discovery modes, affecting how it is detected by central devices. These modes are set using advertisement flags in the advertising packet.

### Discovery Modes  

1. **Non-Discoverable**  
   - Default mode when no advertising is active or when a connection is established.  
   - Cannot be discovered or connected to.  

2. **Limited-Discoverable**
   - Discoverable **for a limited time** to save power.  
   - If no connection is made, the device goes idle.  

3. **General-Discoverable**
   - Advertises **indefinitely** until a connection is established.  

### Advertisement Flags  

These flags indicate the discovery mode and BLE support level. They are combined using bitwise OR (`|`):  

| Bit   | Description |
|-------|------------------------------------------------|
| 0     | Limited discoverable mode (temporary advertising). |
| 1     | General discoverable mode (advertises indefinitely). |
| 2     | Set when the device does not support (or doesn't want to) Bluetooth Classic (BR/EDR). |
| 3     | Set if the device can use both Bluetooth Low Energy (LE) and Classic Bluetooth at the same time (Controller level). |
| 4     | Set if the device can run both Bluetooth Low Energy (LE) and Classic Bluetooth at the same time (Host level). |
| 5-7   | Reserved | Not used. |


We will use the softdevice crate later for Bluetooth. It provides a advertisement builder where we can pass advertisement flags.

For example, to configure the peripheral to advertise indefinitely and indicate that it does not support Bluetooth Classic:

```rust
LegacyAdvertisementBuilder::new()
    .flags(&[
         Flag::GeneralDiscovery,    // 0b0000_0010     (Sets bit 1)
         Flag::LE_Only,             // 0b0000_0100    (Sets bit 2)
   ])
```

## Directed vs Undirected Advertising
Indicates whether the advertisement is meant for a specific central device or any nearby device.

- Undirected: Sent to any nearby central or observer. Used when the peripheral is open to connections from any device.

- Directed: Sent to one specific central, identified by its Bluetooth address. Only that device is allowed to respond.


## Connectable vs Non-Connectable Advertising

This tells whether a central device is allowed to initiate a connection with the peripheral.

- Connectable: The central can send a connection request to the peripheral.

- Non-Connectable: The peripheral only sends advertisements; it will not accept any connection requests.


## Scannable vs Non-Scannable Advertising

Defines whether a central can request extra information from the peripheral via scan request.

- Scannable: The peripheral accepts scan requests and responds with extra information (scan response).

- Non-Scannable: The peripheral does not respond to scan requests.

## Example

In the nrf-softdevice crate, to make your device connectable, scannable, and undirected (i.e. visible to any central nearby), you would write:

```rust
peripheral::ConnectableAdvertisement::ScannableUndirected {
   adv_data: &ADV_DATA,
   scan_data: &SCAN_DATA,
}
```

This tells the SoftDevice to accept connection requests from any central device, respond to scan requests with additional data, and advertise to all nearby devices without targeting a specific one.
