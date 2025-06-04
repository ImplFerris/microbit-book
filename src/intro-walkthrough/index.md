# Project Walkthrough

We have successfully flashed and run our first program, which creates a blinking effect. However, we have not yet explored the code or the project structure in detail. In this section, we will recreate the same project from scratch instead of using the template. I will explain each part of the code and configuration along the way. Are you ready for the challenge?

## Create a Fresh Project

We will start by creating a standard Rust binary project. Use the following command:

```bash
cargo new blinky
```

At this stage, the project will contain the usual files as expected.

```sh
├── Cargo.toml
└── src
    └── main.rs
```

Our goal is to reach the following final project structure:

```sh
├── .cargo
│   └── config.toml
├── Cargo.toml
├── Embed.toml
├── memory.x
└── src
    └── main.rs
```


## Dependencies

We will begin by adding the required dependencies for the project. Update the `Cargo.toml` file with the following entries:

```toml
cortex-m-rt = "0.7.3"
microbit-v2 = "0.15.0"
embedded-hal = "1.0.0"
```

We are using the Board Support Package (BSP) approach, so the [microbit-v2](https://crates.io/crates/microbit-v2) crate provides the board support layer for the micro:bit v2.

We will also take a closer look at the other two dependencies, cortex-m-rt and embedded-hal, in separate sections where I can explain their roles in more detail.

