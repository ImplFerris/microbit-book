# Expanding bind_interrupts Macro

Let's go back to the "bind_interrupts!" macro we used earlier:

```rust
bind_interrupts!(struct Irqs {
    TEMP => temp::InterruptHandler;
});
```

At first glance, the syntax might look strange-especially the struct part. It may seem like we are trying to instantiate a struct, but that's not the case. This is just macro input. The macro internally creates a unit struct named "Irqs" and implements the necessary interrupt binding traits for it.

> Unit struct is an empty struct with no fields; most commonly used as marker. And they have a size of zero bytes.

If you are curious, you can use "cargo expand" to see the generated code. It will look something like this:

```rust
use embassy_nrf::interrupt::typelevel;

struct Irqs;

// Ignoring this: ... impl Copy and Clone for Irqs

#[allow(non_snake_case)]
#[no_mangle]
unsafe extern "C" fn TEMP() {
    <temp::InterruptHandler as typelevel::Handler<typelevel::TEMP,>>::on_interrupt();
}

unsafe impl typelevel::Binding<typelevel::TEMP, temp::InterruptHandler,> for Irqs {
}
```

It creates a unit struct named "Irqs". It also creates a function called TEMP(), which is the actual interrupt handler that the hardware will call when the TEMP interrupt happens. Inside that function, it calls on_interrupt() from temp::InterruptHandler. After that, it implements the Binding trait for the Irqs struct. This tells Embassy that the TEMP interrupt is connected to our handler.

## Marker

If you are wondering why we implement the Binding trait for the Irqs struct when we are not even defining any function inside it, we need to look at the Temp::new function to understand why.

```rust
// let mut temp = Temp::new(p.TEMP, Irqs);

pub fn new(
    _peri: impl Peripheral<P = TEMP> + 'd,
    _irq: impl interrupt::typelevel::Binding<interrupt::typelevel::TEMP, InterruptHandler> + 'd,
) -> Self {
    into_ref!(_peri);

    // Enable interrupt that signals temperature values
    interrupt::TEMP.unpend();
    unsafe { interrupt::TEMP.enable() };

    Self { _peri }
}
```

Wait a minute, the second argument "_irq" is never used. So what is the point of it?

Even though it is not used directly, its type is important. This is how Embassy checks that we have set up a handler for the TEMP interrupt. The type of _irq must implement the Binding trait for TEMP. If it doesn't, the code will not compile.


For example, if you write this: 
```rust
bind_interrupts!(struct Irqs {
    TWISPI0 => twis::InterruptHandler<peripherals::TWISPI0>;
});
```

The macro will generate a Binding trait for the TWISPI0 interrupt, like this:

```rust
// I'm commenting this to keep the Focus on the Binding trait to understand the marker
// #[allow(non_snake_case)]
// #[no_mangle]
// unsafe extern "C" fn TWISPI0() {
//     <twis::InterruptHandler<
//         peripherals::TWISPI0,
//     > as typelevel::Handler<
//         typelevel::TWISPI0,
//     >>::on_interrupt();
// }

unsafe impl typelevel::Binding<typelevel::TWISPI0,twis::InterruptHandler<peripherals::TWISPI0>,> for Irqs {

}
```

But this does not match what Temp::new expects. It expects a Binding for the TEMP interrupt. So the compiler will give an error.


