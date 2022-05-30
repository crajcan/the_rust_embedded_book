## Table of Contents<!-- omit in toc -->
- [1.1 Hardware](#11-hardware)
- [1.2 no_std](#12-no_std)
  - [Hosted Environments](#hosted-environments)
  - [Bare Metal Environments](#bare-metal-environments)
  - [The libstd Runtime](#the-libstd-runtime)
- [Tooling](#tooling)
  - [cargo-binutils](#cargo-binutils)
  - [qemu-system-arm](#qemu-system-arm)
  - [GDB](#gdb)
  - [OpenOCD](#openocd)

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

## Tooling

We'll have to leverage some tooling to develop on a different architecture, and to run and debug on a remote device:

`cargo-binutils` - Cargo subcommands to invoke the LLVM tools shipped with the Rust toolchain.
`qemu-system-arm` - A open source ARM emulator.
`Open OCD (Open On Chip Debugger)` - A debugging interface for the on board debugging hardware.
GDB with ARM support

### cargo-binutils

Includes the LLVM version so `objdump`, `nm`, and `size` and are used for inspecting binaries.

`objdump` - dissassemble an executable into assembly language.

`nm` - dumps the symbol table from a binary file. 

`size` - list the section sizes and total size for provided binary files.

The advantage of using the LLVM versions instead of the GNU binutils is that we can install them with one `rustup` command, and they support all the same architectures as rustc since they both share a backend.

### qemu-system-arm

Allows us to follow along even without the hardware!

### GDB

Since we can't always log to the host console. To use GDB to debug on the remote device, will use GDB's `load` command, which uploads the program to the target hardware. **Check this is true**

### OpenOCD

OpenOCD will act as a translator for GDB to interact with the `ST-Link` debugging hardware on the discovery board. 

OpenOCD runs on the laptop and translates between GDB's TCP/IP based remote debug protocol and ST-LINK's USB based protocol.

It also knows how to interact with the memory-mapped registers 