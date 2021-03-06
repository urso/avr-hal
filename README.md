avr-hal [![Build Status](https://travis-ci.com/Rahix/avr-hal.svg?branch=master)](https://travis-ci.com/Rahix/avr-hal)
=======
`embedded-hal` implementations for AVR microcontrollers.  Based on the register definitions from [`avr-device`](https://github.com/Rahix/avr-device).

## Quickstart
You need nightly rust for compiling rust code for AVR.  Go into `./boards/arduino-leonardo` (or the directory for whatever board you want), and run the following commands:
```bash
# Now you are ready to build your first avr blink example!
cargo +nightly build --example leonardo-blink

# Finally, convert it into a .hex file that you can flash using avr-dude
../../mkhex.sh --debug leonardo-blink

ls -l ../../target/leonardo-blink.hex
```

## Starting your own project
This is a step-by-step guide for creating a new project targeting Arduino Leonardo (`ATmega32U4`).  You can of course apply the same steps for any other microcontroller.

1. Start by creating a new project:
   ```bash
   cargo new --bin avr-example
   cd avr-example
   ```
2. If you're using rustup, you probably want to set an override for this directory, to use the nightly toolchain:
   ```bash
   rustup override set nightly
   ```
3. Copy the target description for your MCU (e.g. `boards/arduino-leonardo/avr-atmega32u4.json`) into your project.
4. Create a file `.cargo/config.toml` with the following content:
   ```toml
   [build]
   target = "avr-atmega32u4.json"

   [unstable]
   build-std = ["core"]
   ```
5. Fill `Cargo.toml` with these additional directives:
   ```toml
   [dependencies]
   # A panic handler is needed.  This is a crate with the most basic one.
   # The `leonardo-panic` example shows a more elaborate version.
   panic-halt = "0.2.0"

   [dependencies.arduino-leonardo]
   git = "https://github.com/Rahix/avr-hal"

   # Configure the build for minimal size
   [profile.dev]
   panic = "abort"
   lto = true
   opt-level = "s"

   [profile.release]
   panic = "abort"
   codegen-units = 1
   debug = true
   lto = true
   opt-level = "s"
   ```
6. Start your project with this basic template:
   ```rust
   #![no_std]
   #![no_main]

   // Pull in the panic handler from panic-halt
   extern crate panic_halt;

   use arduino_leonardo::prelude::*;

   #[arduino_leonardo::entry]
   fn main() -> ! {
       let dp = arduino_leonardo::Peripherals::take().unwrap();

       unimplemented!()
   }
   ```
6. Build with these commands (make sure you're using _nightly_ rust!):
   ```bash
   cargo build
   # or
   cargo build --release
   ```
   and find your binary in `target/avr-atmega32u4/debug/` (or `target/avr-atmega32u4/release`).

## Structure
This repository contains the following components:
* A generic crate containing implementations that can be used chip-independently and macros to create chip-dependent instances of peripheral abstractions.  This crate is named [`avr-hal-generic`](./avr-hal-generic).
* HAL crates for each chip in `chips/`.  These make use of `avr-hal-generic` to create chip-specific definitions.
* Board Support Crates for popular hardware in `boards/`.  They, for the most part, just re-export functionality from the chip-HAL, with the names that are printed on the PCB.

## Status
The following peripherals are supported in `avr-hal-generic`:
- [x] A spinning delay implementation
- [x] `PORTx` peripherals as digital IO (v2)
- [x] A TWI based I2C implementation
- [X] SPI primary-mode implementation

### HAL Status
The chip-HAL crates currently support the following peripherals:
* [`atmega2560-hal`](./chips/atmega2560-hal)
  - [x] Spinning Delay
  - [x] `PORTA`, `PORTB`, `PORTC`, `PORTD`, `PORTE`, `PORTF`, `PORTG`, `PORTH`, `PORTJ`, `PORTK`, `PORTL` as digital IO
  - [x] `USART0`, `USART1`, `USART2`, `USART3` for serial communication
  - [x] I2C using `TWI`
  - [x] SPI
  - [x] ADC (no differential channels yet)
* [`atmega328p-hal`](./chips/atmega328p-hal)
  - [x] Spinning Delay
  - [x] `PORTB`, `PORTC`, `PORTD` as digital IO (v2)
  - [x] `USART0` for serial communication
  - [x] I2C using `TWI`
  - [x] SPI
  - [x] ADC
* [`atmega32u4-hal`](./chips/atmega32u4-hal)
  - [x] Spinning Delay
  - [x] `PORTB`, `PORTC`, `PORTD`, `PORTE`, `PORTF` as digital IO (v2)
  - [x] `USART1` for serial communication
  - [x] I2C using `TWI`
  - [x] SPI
  - [x] ADC (no differential channels yet)
* [`attiny85-hal`](./chips/attiny85-hal)
  - [x] Spinning Delay
  - [x] `PORTB` as digital IO (v2)
* [`attiny88-hal`](./chips/attiny88-hal)
  - [x] Spinning Delay
  - [x] `PORTA`, `PORTB`, `PORTC`, `PORTD` as digital IO
  - [x] I2C using `TWI`
  - [x] SPI

### Supported Hardware
In `boards/` there are crates for the following hardware.  Please note that this project is in no way affiliated with any of the vendors.

* [Arduino Leonardo](./boards/arduino-leonardo)
  - [Website](https://www.arduino.cc/en/Main/Arduino_BoardLeonardo)
* [Arduino Uno](./boards/arduino-uno)
  - [Website](https://store.arduino.cc/usa/arduino-uno-rev3)
* [Arduino Mega 2560](./boards/arduino-mega2560)
  - [Website](http://arduino.cc/en/Main/ArduinoBoardMega2560)
* [Adafruit Trinket (3V3 or 5V)](./boards/trinket) (**not** PRO!)
  - [Website](https://learn.adafruit.com/introducing-trinket)
* [BigAVR 6](./boards/bigavr6)

## Disclaimer
This project is not affiliated with either Microchip (former Atmel) nor any of the Vendors that created the boards supported in this repository.

## License
*avr-hal* is licensed under either of

 * Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
 * MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.
