
# Interrupt Handler for the TEMP Peripheral

The first question you might ask is: Why do we even need an interrupt handler here? Why are we talking about this now?

Well, the temperature sensor is just another peripheral. If we want to measure temperature, the CPU has to ask the sensor to start a measurement and then wait for the result. But the measurement takes a bit of time, and it's wasteful for the CPU to sit idle and keep checking if the value is ready (this is called polling). A better approach is to let the sensor interrupt us when it's ready.

Setting up interrupts involves multiple steps, but for our case we'll only focus on what's directly relevant.

## First, we bound the TEMP interrupt to its handler

The embassy-nrf crate provides a macro called "bind_interrupts!" that helps us connect specific "Interrupt" to their corresponding handler.  

The general usage looks like this:

```rust
bind_interrupts!(struct Irqs{
    INTERRUPT_NAME => INTERRUPT_HANDLER;
    INTERRUPT_NAME2 => INTERRUPT_HANDLER2;
});
```

In our case, we bind the "TEMP" interrupt to handler "temp::InterruptHandler" which is also provided by the embassy-nrf.

```rust
bind_interrupts!(struct Irqs {
    TEMP => temp::InterruptHandler;
});
```
Here, we are basically telling: "Hey, if any interrupt comes from the TEMP peripheral, let temp::InterruptHandler take care of it."

## Then, we initialized the Temp struct
We created the Temp driver by passing in the p.TEMP peripheral and the Irqs struct we just defined:
```rust
let mut temp = Temp::new(p.TEMP, Irqs);
```
You might be thinking - "Wait, that Irqs thing doesn't look like a unit struct". And you're right, it doesn't. But trust me, it is one. The macro we used earlier creates that unit struct. I will shortly show you what it looks like after the macro expands.

## Finally, we read the temperature asynchronously
Now we just needed to read the temperature like this:

```rust
let value = temp.read().await;
```

> From nRF52833 doc: TEMP is started by triggering the START task. When the temperature measurement is completed, a DATARDY event will be generated and the result of the measurement can be read from the TEMP register.

This function internally sends a request to the sensor to start the temperature measurement. Then it enables the interrupt so that the sensor could notify us when the data is ready.

It waits for the result asynchronously. When the sensor finished the measurement, it triggers an interrupt. That interrupt is handled by temp::InterruptHandler, which woke up the read() function so it could go ahead, read the temperature value.

---

## Temperature Interrupt Handler

If we look at the definition of the temp::InterruptHandler struct, we can see that it implements the Handler trait. The main part is the "on_interrupt" function, which tells what should happen when the TEMP interrupt is received.

Like we said earlier, it does not do much. It simply informs Embassy to wake up the read() function that was waiting for the sensor.

```rust
impl interrupt::typelevel::Handler<interrupt::typelevel::TEMP> for InterruptHandler {
    unsafe fn on_interrupt() {
        let r = pac::TEMP;
        r.intenclr().write(|w| w.set_datardy(true));
        WAKER.wake();
    }
}
```
This function first disables the interrupt so it does not fire again. Then it calls WAKER.wake() to resume the async task that was waiting for the temperature value. That allows the read() function to continue and read the result







