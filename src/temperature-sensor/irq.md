# Interrupt Request (IRQ)

An Interrupt Request (IRQ) is a hardware signal triggered by a peripheral (e.g., sensor, timer). This signal immediately captures the CPU's attention, pausing its current tasks so it can handle the event through a specialized routine called an Interrupt Service Routine (ISR). Upon completion of the ISR, the CPU restores its previous context and resumes the original task. 

Without IRQs, the CPU would need to continuously check (poll) each peripheral to see if something happened. This wastes time and energy, especially when most of the time, nothing is changing.

## Why Use IRQ Instead of Polling?

Imagine this analogy: you are playing your favorite video game, fully focused and trying to defeat the final boss. At the same time, you are expecting a friend to visit.

**Polling:** You keep pausing the game every few seconds to check the door. It is distracting, inefficient, and ruins the game play.

**IRQ (Doorbell):** You install a doorbell. When your friend arrives, they press it. You hear the ring, quickly pause the game, open the door, then return to your game exactly where you left off.

---

In this section, we will not go into the full details or all the steps involved in defining your own ISR. That requires its own dedicated chapter. For now, we will just have a gentle introduction to the interrupt macro and handler support provided by the HAL.
