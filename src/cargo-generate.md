# Project Template with `cargo-generate`

"cargo-generate is a developer tool to help you get up and running quickly with a new Rust project by leveraging a pre-existing git repository as a template."

Read more about [here](https://github.com/cargo-generate/cargo-generate).
 
## Prerequisites

Before starting, ensure you have the following tools installed:

- [cargo-generate](https://github.com/cargo-generate/cargo-generate) for generating the project template.

You can install `cargo-generate` using the following command:
```sh
cargo install cargo-generate
```

## Step 1: Generate the Project

I have created a project template for the microbit to make project setup and learning easier. Run the following command to generate a new project from the template:

```sh
cargo generate --git https://github.com/ImplFerris/mb2-template.git
```

This will prompt you to answer a few questions:
Project name: Name your project.

