# Introduction to nRF HAL

Before we move on to other examples, let us first introduce the Hardware Abstraction Layer (HAL) for the nRF51, nRF52, and nRF91 families of microcontrollers.

As you may already know, the micro:bit v2 uses the Nordic [nRF52833](https://www.nordicsemi.com/Products/nRF52833) microcontroller. 

Until now, we have worked with BSP-level crates. Now, we will go one layer deeper into the HAL. For this purpose, we will use the nrf-hal, which provides support for the nRF52833 as well.

You can refer to the [nrf-hal GitHub repository](https://github.com/nrf-rs/nrf-hal) for more details. It also includes examples for various use cases.

## Rewrite Blinky

To keep things simple, let us rewrite the blinky example using `nrf-hal`.

If you are creating a project from scratch, you would typically add `nrf52833-hal` as a dependency manually. However, we will use our template that already includes it. In the template's `Cargo.toml`, you will find a line like this:

```toml
nrf52833-hal = "0.18.0" # Version might be different in the template
```


## Create Project from template

To generate a new project using the template, run the following command:

```sh
cargo generate --git https://github.com/ImplFerris/mb2-template.git
```

When prompted for a project name, enter something like `led-blinky` (If you already have a project with this name, use a different name or place HAL-based projects in a separate folder, like I do.)

When it prompts to select "BSP" or "HAL", select the option "HAL".

