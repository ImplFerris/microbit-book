# SoftDevice Firmware

Before we run our Bluetooth program, we need to flash a special firmware called the SoftDevice onto the micro:bit. This firmware is provided by Nordic Semiconductor, the maker of the nRF52833 chip used in the micro:bit v2.

For our chip, we'll use SoftDevice S113, which is a memory-efficient Bluetooth Low Energy (BLE) protocol stack. It supports up to 4 simultaneous BLE connections as a peripheral and can also broadcast data.

## Download 

1. Go to the Nordic Semi download page:
[https://www.nordicsemi.com/Products/Development-software/S113/Download](https://www.nordicsemi.com/Products/Development-software/S113/Download)

2. Download the package. In my case, I got a file named DeviceDownload.zip.

3. Extract DeviceDownload.zip and you'll find another file: s113_nrf52_7.3.0.zip.

4. Extract that zip too. Inside, you'll see something like this:

```
.
├── s113_nrf52_7.3.0_API
├── s113_nrf52_7.3.0_license-agreement.txt
├── s113_nrf52_7.3.0_release-notes.pdf
└── s113_nrf52_7.3.0_softdevice.hex

1 directory, 3 files
```

We are interested in the .hex file. This is a special file format ([Intel HEX](https://tech.microbit.org/software/hex-format/)) that contains binary firmware data in a readable text format.


## Flash the SoftDevice

To flash the firmware onto the micro:bit, run this command:
```sh
probe-rs download s113_nrf52_7.3.0_softdevice.hex --binary-format Hex --chip nRF52833_xxAA
```

Just keep in mind that you'll need to repeat this step whenever you come back to the BLE exercises, as other exercises might have overwritten the SoftDevice firmware.


## Update memory.x file 

After flashing the SoftDevice, we must make sure our Rust program does not overwrite the memory region used by the SoftDevice.

To do that, we update the [memory.x](/intro-walkthrough/are-we-there-yet.html#memoryx) linker script. This file defines which regions of memory the Rust compiler can use for your application.

### Original memory.x file

```
MEMORY
{
  FLASH : ORIGIN = 0x00000000, LENGTH = 512K
  RAM : ORIGIN = 0x20000000, LENGTH = 128K
}
```

### Updated memory.x file

```
MEMORY
{
  /* NOTE 1 K = 1 KiBi = 1024 bytes */
  MBR                               : ORIGIN = 0x00000000, LENGTH = 4K
  SOFTDEVICE                        : ORIGIN = 0x00001000, LENGTH = 112K
  FLASH                             : ORIGIN = 0x0001C000, LENGTH = 396K
  RAM                               : ORIGIN = 0x20003410, LENGTH = 117744
}
```

This tells the linker to start your program after the SoftDevice's reserved flash and RAM space, preventing memory conflicts at runtime.

## Where Do These Values Come From?

These values are based on the pdf document provided along with the hex file  (s113_nrf52_7.3.0_release-notes.pdf):

### Flash Memory

The SoftDevice uses the first 112 KB (0x1C000) of flash memory, starting from 0x00001000. That means our app should start after 0x0001C000, leaving us with 512 KB - 112 KB - 4 KB (MBR) = 396 KB of usable flash.

### RAM Calculation

The minimum RAM required by the SoftDevice depends on how we configure it. According to the release notes, it needs at least 4.4 KB (0x1198), and it can use around 1.8 KB (0x700) for the call stack (in the worst case). But we actually don't need to guess or manually calculate the exact amount to reserve.

The nice part is that the nrf-softdevice crate will tell us exactly how much RAM it needs when the program runs.

Here's the trick: start with a small RAM reservation - for example, try setting the program's RAM start address to 0x20001fa8 (so you're reserving about 8 KB initially). When you flash and run the program, you'll get a log like this:

```sh
softdevice RAM: 13328 bytes
```

So in this case, it needs 13328 bytes, which is 0x3410 in hex. That's why in the final memory.x, we set the program RAM start at 0x20003410. The total RAM on the chip is 128 KB = 131072 bytes, so the remaining RAM for your app becomes:

```sh
# remaining_ram = total_ram - softdevice_ram
remaining_ram = 131072 - 13328 = 117744 bytes
```
That's the usable RAM your program gets after the SoftDevice takes its share.

> Special thanks to Dario Nieuwenhuis for helping me understand the RAM calculation for the SoftDevice.

NOTE: You might need to adjust the RAM start and length in memory.x if the SoftDevice configuration changes later. But no need to worry - the nrf-softdevice crate will tell you exactly how much RAM it needs at runtime. We can then adjust them and re-run the program.
```
RAM                               : ORIGIN = 0x20003410, LENGTH = 117744
```

The nrf-softdevice crate will also warn you if you allocate more memory to the SoftDevice than necessary. 
