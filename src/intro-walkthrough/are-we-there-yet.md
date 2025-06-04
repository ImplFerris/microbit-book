# Are we there yet?

We have successfully been able to build the program, but are we ready to flash it (write it into the micro:bit's persistent memory and run it)? Not yet but we are close. There are a few more steps we need to complete before that.

If you try running `cargo embed` now, you'll see this error:

```sh
...other warnings...
WARN probe_rs::flashing::loader: No loadable segments were found in the ELF file.
Error No loadable segments were found in the ELF file.
```

This error means that the compiler created a file (ELF file) that doesn't have any actual code or data for the flasher to write into the micro:bit. In other words, the file is empty from the flasher's point of view.

Why did this happen? Because the compiler doesn't know where in memory to place the program. Even though we are using the cortex-m-rt crate (which gives us startup code and other support), the linker still needs to know how the micro:bit's memory is laid out. Without this information, it skips putting the actual program into the output file.

## Fixing the Error

To solve this, we need to tell the compiler to use a special script called link.x. This script is provided by the cortex-m-rt crate and tells the compiler how to place code and data into memory.

Update the `.cargo/config.toml` with this:

```toml
[target.thumbv7em-none-eabihf]
rustflags = ["-C", "link-arg=-Tlink.x"]
```

This line adds a flag that tells the compiler: "Please use link.x to figure out the memory layout."

## memory.x
According to the cortex-m-rt crate documentation, we need a file called memory.x to define the memory layout of the microcontroller. This file tells the linker where the flash and RAM start, and how large they are.

So... where is it?

If you try to flash the device at this stage itself, the program should get flashed and run on the micro:bit. That's because the memory.x file is automatically included from the nrf52833-hal crate. You can see it here: [https://github.com/nrf-rs/nrf-hal/blob/master/nrf52833-hal/memory.x](https://github.com/nrf-rs/nrf-hal/blob/master/nrf52833-hal/memory.x)

However, it's better to define the memory.x file in our own project to avoid any kind of ambiguity. So in the project root folder, create a file named memory.x with the following content:

```
MEMORY
{
  FLASH : ORIGIN = 0x00000000, LENGTH = 512K
  RAM : ORIGIN = 0x20000000, LENGTH = 128K
}
```

This tells the linker where the flash and RAM start, and how much memory is available.

- `ORIGIN = 0x00000000` means the flash memory begins at address `0x00000000`.
- `LENGTH = 512K` means the flash memory size is 512 kilobytes (512 × 1024 bytes).
- `ORIGIN = 0x20000000` means the RAM starts at address `0x20000000`.
- `LENGTH = 128K` means the RAM size is 128 kilobytes.

These addresses and sizes are based on the nRF52833 document.

---

At this point, your project folder should look like this:

```
├── .cargo
│   └── config.toml
├── Cargo.toml
├── memory.x
└── src
    └── main.rs
```


## Flash it 

Now try building and flashing your program:

```sh
cargo flash

# OR

cargo embed
```
This time, the ELF file will contain valid loadable segments, and the flasher will be able to write it into the micro:bit's flash memory.

If all goes well, your program should now be running on the micro:bit! You should see the blinking effect on the first LED.


## Reference

- [Memory map](https://docs.nordicsemi.com/bundle/ps_nrf52833/page/memory.html#ariaid-title4)