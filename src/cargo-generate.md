# Micro:bit Project Template with `cargo-generate`

To simplify project setup and learning for the micro:bit, I've created a reusable project template. We'll use the `cargo-generate` tool to get started.

> What is `cargo-generate`?  
> `cargo-generate` is a tool that helps you quickly create new Rust projects using pre-made templates, avoiding boilerplate setup and code.


You can learn more about it [here](https://github.com/cargo-generate/cargo-generate).

## Prerequisites

Before installing `cargo-generate`, make sure you have `libssl-dev` installed. 

On Ubuntu or Debian-based systems, run:

```bash
sudo apt install libssl-dev
```

Then, install cargo-generate with:

```bash
cargo install cargo-generate
```

## Step 1: Generate a New Project

Once `cargo-generate` is installed, you can generate a new project using the following command:

```sh
cargo generate --git https://github.com/ImplFerris/mb2-template.git --rev 88d339b
```

> Note: I have included the specific rev (revision) value in the cargo generate command to ensure the setup is reproducible. Without it, future changes to the template might break compatibility with this tutorial.


You will be prompted to enter a project name. 

You will then prompted to choose "BSP" or "HAL"; 

After that, a new directory with that name will be created. Navigate into it:

```sh
cd your-project-name
```

Now you're ready to build and run your micro:bit project.

To flash and run the code on the micro:bit, use:

```sh
cargo embed
```
