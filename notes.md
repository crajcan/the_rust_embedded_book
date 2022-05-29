## Table of Contents<!-- omit in toc -->
- [1.1 Hardware](#11-hardware)
- [1.2 no_std](#12-no_std)
  - [Hosted Environments](#hosted-environments)
  - [Bare Metal Environments](#bare-metal-environments)
  - [The libstd Runtime](#the-libstd-runtime)

## 1.1 Hardware

The book will have us working iwth the STM32F3DISCOVERY (the "F3") board. 

The board includes an STM32F303VCT6 microcontroller with:
- 256 kiB of "Flash" memory
- 48 KiB of RAM
- integrated peripherals
  - timers
  - _I2C_ (Inter-Integrated Circuit), A synchronous communcation bus for connecting integrated circuits
  - _SPI_ (Serial Peripheral Interface), used for short distance communcations. Used often with SD cards and LCD screens. 
  - _USART_ (Universal Synchronouse and Asynchronouse receiver-transmitter) a serial interface device that can be programmed to communcate asynchonously 
- _General Purpose Input Output_ (GPIO) and other types of pins
- A USB interface labeled "USB USER"
- An accelerometer
- A magnetometer (A device for measuring the direction, strenght or relative change of a magnetic field)
- A gyroscope
- 8 user LEDs
- A second microcontroller for debugging.

## 1.2 no_std

The two general Embedded Programming classifications

### Hosted Environments

You are provided a System Interface (POSIX) that gives you primitives for interacting with various systems such as file systems, networking, memory management, threads, etc. With this inteface available, Rust (among other langauges) are able to provide you with their standard library for interacting with the system. This is like coding in a special purpouse PC environment.

### Bare Metal Environments

No code has been loaded before your program. With no OS-provided software, we cannot load the standard library (`libstd`). Thus, your program can only use the hardware (bare metal) to run. Use `no_std` to prevent Rust from trying to load the standard library. `libcore` provides the platform-agnostic parts of `libstd`. We will use this instead of the full standard library for bare metal applications. `#![no_std]` is the crate level attribute that indicates the crate will link to the core-crate instead of the std-crate.

Since `libcore` makes no assumptions about the system the program will run on, it can be used for any kind of `bootstrapping` (level 0) code like bootloaders, firmware, or kernals. 
### The libstd Runtime

`libstd` provides not only a common way for accessing OS abstractions, but also provides a runtime. The runtime handles certain tasks before spawning the main thread and invoking the program's main function, such as setting up stack overflow protection and processing command line arguments.

