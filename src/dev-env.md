# Development Environment

I am going to assume you already have Rust installed on your machine and that you know the basics. If not, it might be a bit challenging to follow along. I highly recommend learning the basics of Rust first, then coming back to this. 

## probe-rs
probe-rs is a toolkit for working with embedded ARM and RISC-V devices. It supports flashing firmware, debugging programs, and printing logs via various debug probes.

The project includes tools like:
- cargo-flash -quickly flash firmware to a target
- cargo-embed - open a full RTT terminal with support for multiple channels and command input

We will be using this to flash (i.e writing the code into the device and running it) our program onto the microbit and run it; also for debugging purposes. 

You can find more information [here](https://probe.rs/). Please see the [installation guide](https://probe.rs/docs/getting-started/installation/) for setup instructions.

You can confirm the installation was successful by running this command:
```bash
cargo embed --version
```

## Cross compilation target

Since the micro:bit runs on an ARM Cortex-M processor, we need to compile our Rust code for that architecture. This requires setting up a specific compilation target for cross-compiling.

For the micro:bit v2, the correct target is:

```text
thumbv7em-none-eabihf
```

You can add this target using Rust's built-in toolchain manager:
```sh
rustup target add thumbv7em-none-eabihf
```

Once added, you can specify this target when building or flashing your project. For example:
```sh
cargo build --release --target thumbv7em-none-eabihf
```

Or it will be used automatically when running tools like `cargo embed`, as long as your project is configured correctly(this part we will get into later chapter).

