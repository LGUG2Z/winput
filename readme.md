[![Documentation](https://docs.rs/winput/badge.svg)](https://docs.rs/winput/)
[![Crates.io](https://img.shields.io/crates/v/winput.svg)](https://crates.io/crates/winput)

`winput` is a high-level interface to *Windows*' input system.

## Target

This crate aims to be low-level and straightforward enough to be used as a backend for other, more general crates of the genre. For this purpose, the "minimal" feature disables most of the stuff that is not really part of *Windows*' input system (things like [`Keylike`], for example, that are mostly there for convenience).

## What is left to do?

`winput` does not currently support any devices other than the mouse and the keyboard. I haven't really looked into how those work so if you know anything, feel free to submit an issue or a pull request!

## Examples

The [`Keylike`] structure allows you to synthesize keystrokes on objects that can be used as keys.

```rust
use winput::{Vk, Button};

// Synthesize keystrokes from a Virtual-Key Code
winput::press(Vk::Shift).unwrap();    // press the shift key
winput::send(Vk::A).unwrap();         // press then release the A key
winput::release(Vk::Shift).unwrap();  // release the shift key

// Synthesize keystrokes from characters
winput::send('F').unwrap();
winput::send('O').unwrap();
winput::send('O').unwrap();

// Synthesize keystrokes from mouse buttons
winput::send(Button::Left).unwrap();

// You can synthesize keystrokes for the characters of a string
winput::send_str("Hello, world!");
```

The [`Mouse`] structure can be used to manipulate the mouse.

```rust
use winput::Mouse;

// Retrieve the position of the mouse.
let (x, y) = Mouse::position().unwrap();

// Set the mouse position
//  ... in screen coordinates
Mouse::set_position(10, 10).unwrap();
//  ... in normalized absolute coordinates
Mouse::move_absolute(0.5, 0.5).unwrap();
//  ... relatively to the current cursor's position
Mouse::move_relative(100, 50).unwrap();

// Rotate the mouse wheel (vertically)
Mouse::scroll(1.5).unwrap();
//  ... or horizontally
Mouse::scrollh(-1.5).unwrap();
```

For more complicated input patterns, the [`Input`] structure can be used.

```rust
use winput::{Input, Vk, Action, MouseMotion};

// There is multiple ways to create an `Input`:
let inputs = [
    // ... from a character
    Input::from_char('a', Action::Press).unwrap(),
    // ... from a Virtual-Key Code
    Input::from_vk(Vk::A, Action::Release),
    // ... from a mouse motion
    Input::from_motion(MouseMotion::Relative { x: 100, y: 100 }),

    // additional constructors are available
];

let number_of_inputs_inserted = winput::send_inputs(&inputs);

assert_eq!(number_of_inputs_inserted, 3);
```

With the `events` feature, keyboard keystrokes and mouse inputs can be retreived. See the [`Handler`] trait for more information.

```rust
use winput::events::{self, Handler};
use winput::{Vk, Action};

struct MyHandler;
impl Handler for MyHandler {
    fn keyboard(&self, vk: Vk, _scan_code: u32, action: Action) {
        if action == Action::Press {
            println!("{:?}", vk);
        }
    }
}

fn main() {
    // Subscribe the handler to the Windows's global message loop.
    let h = events::subscribe_handler(MyHandler);
    
    std::thread::sleep(std::time::Duration::from_secs(60));

    // Unsubscribe the handler.
    events::unsubscribe_handler(h);
}
```

[`Keylike`]: https://docs.rs/winput/latest/winput/trait.Keylike.html
[`Input`]: https://docs.rs/winput/latest/winput/struct.Input.html
[`Mouse`]: https://docs.rs/winput/latest/winput/struct.Mouse.html
[`Handler`]: https://docs.rs/winput/latest/winput/event_loop/trait.Handler.html