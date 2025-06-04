# What Are Interrupts?
In embedded systems, an interrupt is a special signal sent to the processor to get its immediate attention. Think of it like someone tapping you on the shoulder while you are focused on a task.

For example, an interrupt might be triggered when:

- A button is pressed

- A timer expires

- Data arrives on a serial port

When this happens, the processor temporarily pauses its current task, runs a special function called an interrupt handler  and then resumes what it was doing before. This allows the system to respond to important events immediately, even if it is in the middle of doing something else.

## Resources

-  The Embedded Rust Book's Interrupts section: [https://docs.rust-embedded.org/book/start/interrupts.html](https://docs.rust-embedded.org/book/start/interrupts.html)