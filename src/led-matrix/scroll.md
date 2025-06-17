# Scrolling Effect

In this section, we will create a scrolling effect for a single character. The character will scroll from right to left,disappear off the edge,and then reappear from the right again.

There is another BSP crate called [`microbit-bsp`](https://github.com/lulf/microbit-bsp) that provides built-in support for scrolling text. However, it uses the Embassy framework and asynchronous programming (`async`). Since we have not yet introduced Embassy or async concepts, we will avoid using that crate for now.

So for now, we will stick with the `microbit-v2` crate and implement the scrolling logic ourselves.

## Logic

You already know how to turn on the LED matrix using a 2D array. Now, try to think of a way to create a scrolling effect using that knowledge. There are multiple ways to do this. I encourage you to come up with your own logic first.

Below, I will show you one possible way to implement it. Keep in mind that this is just one approach, and it may not be the most efficient or elegant solution.

## The Full code

```rust
#![no_std]
#![no_main]

use embedded_hal::delay::DelayNs;
use microbit::{board::Board, display::blocking::Display, hal::timer::Timer};
use cortex_m_rt::entry;

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

#[entry]
fn main() -> ! {
    let board = Board::take().unwrap();
    let mut timer = Timer::new(board.TIMER0);
    let mut display = Display::new(board.display_pins);

    // 5x5 representation of 'R'
    let r_char = [
        [1, 1, 1, 0, 0],
        [1, 0, 0, 1, 0],
        [1, 1, 1, 0, 0],
        [1, 0, 1, 0, 0],
        [1, 0, 0, 1, 0],
    ];

    let mut offset = 0;

    loop {
        let mut frame = [[0; 5]; 5];

        for row in 0..5 {
            for col in 0..5 {
                let char_col = col as isize + offset - 4;

                if char_col >= 0 && char_col < 5 {
                    frame[row][col] = r_char[row][char_col as usize];
                } else {
                    frame[row][col] = 0;
                }
            }
        }

        display.show(&mut timer, frame, 500);
        timer.delay_ms(100);

        offset += 1;
        if offset > 8 {
            offset = 0;
        }
    }
}
```

We use a variable called `offset` to control which part of the character we show on the screen. As offset increases, the character moves to the left.

### Scrolling with Offset
Our LED matrix is 5 columns wide. To scroll the character in from the right, move it fully across, and have it scroll out to the left, we need to allow for more than 5 steps.

Let's break it down:

- The character starts completely off-screen on the right.

- Then, it enters one column at a time.

- It becomes fully visible in the center.

- Finally, it moves out one column at a time to the left until it disappears.

We want to cover this full path:

```
[off-screen right] --> [entering display] --> [fully visible] --> [leaving display] --> [off-screen left]
```

This takes a total of 9 steps (offsets from 0 to 8):

| Offset | What happens on screen                                 | How many columns of character are shown? | Character's position relative to the display |
| ------ | ------------------------------------------------------ | ---------------------------------------- | -------------------------------------------- |
| **0**  | **First column** of the character appears at far right | 1                                        | Character is mostly off-screen to the right  |
| **1**  | First **two columns** appear                           | 2                                        | Character slides in more                     |
| **2**  | First three columns appear                             | 3                                        | Half-visible                                 |
| **3**  | First four columns appear                              | 4                                        | Almost fully visible                         |
| **4**  | Entire character is fully visible                      | 5                                        | Just as how it appears normally              |
| **5**  | First column starts disappearing from the left         | 4                                        | Character starts sliding off to the left     |
| **6**  | Only middle and right side remain visible              | 3                                        | More of character has exited                 |
| **7**  | Only last part of the character remains                | 2                                        | Nearly gone                                  |
| **8**  | Character is completely gone                           | 0                                        | Fully off-screen to the left                 |



### Creating the Frame

We create a new 5x5 frame that we will send to the display. For each LED in the frame:

We calculate which column of the character (r_char) should be shown in that position. This is done with:
```rust
let char_col = col as isize + offset - 4;
```

This shifts the character slowly to the left.

We check if char_col is in the valid range (0 to 4). If yes, we copy that pixel from the character. If not, we set it to 0 (LED off).

### Loop and Animate
We keep repeating this in a loop:

- Show the current frame for 500 ms then wait 100 ms

- Increase the offset

- Reset offset back to 0 after it reaches 9

This gives the illusion that the character is scrolling from right to left and disappearing, then reappearing again.

## offset, led matrix "col", char_col relation

Let's look at how these values change as the offset increases to understand it better.

**offset = 0**

Only the first column of R (index 0) is visible at the rightmost column.

char_col = col + offset - 4 = col - 4


| `col`        | 0  | 1  | 2  | 3  | 4 |
| ------------ | -- | -- | -- | -- | - |
| `char_col`   | -4 | -3 | -2 | -1 | 0 |



```
. . . . #
. . . . #
. . . . #
. . . . #
. . . . #
```
Note: Using 0s and 1s directly can make it harder to see the shape clearly. So, in this illustration, the `#` symbol shows where an LED is turned on (i.e value 1). The dots (.) represent LEDs that are off (value 0). 


**offset = 1**

Columns 3 and 4 show character columns 0 and 1 respectively.

char_col = col + 1 - 4 = col - 3


| `col`        | 0  | 1  | 2  | 3 | 4 |
| ------------ | -- | -- | -- | - | - |
| `char_col`   | -3 | -2 | -1 | 0 | 1 |

```
. . . # #
. . . # .
. . . # #
. . . # .
. . . # .

```


**offset = 4**

We will skip to the case where the offset is 4. At this point, the full character is completely visible on the display.


char_col = col + 4 - 4 = col

This means char_col and col are equal, so the frame array directly matches the original character array.

| `col`        | 0 | 1 | 2 | 3 | 4 |
| ------------ | - | - | - | - | - |
| `char_col`   | 0 | 1 | 2 | 3 | 4 |


```
# # # . .
# . . # .
# # # . .
# . # . .
# . . # .

```

**offset = 5**

char_col = col + 5 - 4 = col + 1


| `col`        | 0 | 1 | 2 | 3 | 4 |
| ------------ | - | - | - | - | - |
| `char_col`   | 1 | 2 | 3 | 4 | 5 |


```
# # . . .
. . # . .
# # . . .
. # . . .
. . # . .
```

Now, each char_col is one more than its corresponding col. When char_col becomes 5, it goes out of bounds (since our array only has indices from 0 to 4).

To prevent this, we add a bounds check. If char_col is outside the valid range, we fill that column with 0s. This creates the vanishing effect as the character scrolls out.

```
if char_col >= 0 && char_col < 5 {
    frame[row][col] = r_char[row][char_col as usize];
} else {
    frame[row][col] = 0;
}
```

## The Full code
```rust
#![no_std]
#![no_main]

use embedded_hal::delay::DelayNs;
use microbit::{board::Board, display::blocking::Display, hal::timer::Timer};
use cortex_m_rt::entry;

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}

#[entry]
fn main() -> ! {
    let board = Board::take().unwrap();
    let mut timer = Timer::new(board.TIMER0);
    let mut display = Display::new(board.display_pins);

    // 5x5 representation of 'R'
    let r_char = [
        [1, 1, 1, 0, 0],
        [1, 0, 0, 1, 0],
        [1, 1, 1, 0, 0],
        [1, 0, 1, 0, 0],
        [1, 0, 0, 1, 0],
    ];

    let mut offset = 0;

    loop {
        let mut frame = [[0; 5]; 5];

        for row in 0..5 {
            for col in 0..5 {
                let char_col = col as isize + offset - 4;

                if char_col >= 0 && char_col < 5 {
                    frame[row][col] = r_char[row][char_col as usize];
                } else {
                    frame[row][col] = 0;
                }
            }
        }

        display.show(&mut timer, frame, 500);
        timer.delay_ms(100);

        offset += 1;
        if offset > 8 {
            offset = 0;
        }
    }
}
```

## Clone the existing project
You can also clone (or refer) project I created and navigate to the `led-scroll` folder.

```sh
git clone https://github.com/ImplFerris/microbit-projects
cd microbit-projects/bsp/led-scroll
```


## Flash

You can flash the program into the micro:bit.

```sh
cargo flash
```
