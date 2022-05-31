## Table of Contents<!-- omit in toc -->
- [1.1 Hardware](#11-hardware)
- [1.2 no_std](#12-no_std)
  - [Hosted Environments](#hosted-environments)
  - [Bare Metal Environments](#bare-metal-environments)
  - [The libstd Runtime](#the-libstd-runtime)
- [1.3 Tooling](#13-tooling)
  - [cargo-binutils](#cargo-binutils)
  - [qemu-system-arm](#qemu-system-arm)
  - [GDB](#gdb)
  - [OpenOCD](#openocd)
- [2.1 QEMU](#21-qemu)
  - [Cross Compiling](#cross-compiling)
  - [Inspecting](#inspecting)
  - [Running](#running)
  - [Debugging](#debugging)

## 1.1 Hardware

The book will have us working iwth the STM32F3DISCOVERY (the "F3") board. 

The board includes an STM32F303VCT6 microcontroller with:
- 256 kiB of "Flash" memory
- 48 KiB of RAM
- integrated peripherals - timers
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

## 1.3 Tooling

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

## 2.1 QEMU

We will write a program for the LM3S6965, a Cortex-M3 microcontroller, in `chapter_2/app`.

In `chapter_2/app/main.rs`, we declare `#![no_std]` as above.

We then declare `#![no_main]` which prevents the usual main interface from being loaded, since we used `no_std`.

`use panic_halt as _;` declares a panic handler.

`fn main() -> !` specifies `#main` as a _divergent function_, one that never returns, since our program is the only process running on the target.

### Cross Compiling

If we use rustup to add a compilation target to the tool chain, we can then edit `chapter_2/app/.cargo/config.toml` to set the default compilation target, in this case: `thumbv7m-none-eabi`.

### Inspecting

We can use `cargo-readobj` from cargo-binutils to print the _ELF headers_ from the binary:

```
cargo readobj --bin app -- --file-headers
```

ELF stands for `Extensible Linking Format`, It is a standard file format for executables. The ELF headers, among other things, specify the target operating system (`OS/ABI`) and system architecture (`Machine`), in this case `ARM`.

`cargo-size` can print the size of the linker sections of the binary:

```
cargo size --bin app --release -- -A
```

Which outputs:

```
app  :
section              size        addr
.vector_table        1024         0x0
.text                 644       0x400
.rodata                 0       0x684
.data                   0  0x20000000
.bss                    0  0x20000000
.uninit                 0  0x20000000
.debug_loc            346         0x0
.debug_abbrev        1621         0x0
.debug_info          7148         0x0
.debug_aranges        792         0x0
.debug_ranges        1512         0x0
.debug_str          10884         0x0
.debug_pubnames      3247         0x0
.debug_pubtypes       254         0x0
.ARM.attributes        50         0x0
.debug_frame         1720         0x0
.debug_line          8603         0x0
.comment              109         0x0
Total               37954
```

`.text`: contains the program instructions
`.rodata`: contains constants like strings
`.data`: contains statically allocated variables whose initial values are not zero.
`bss`: contains statically allocated variables whose initalize values are zero.A
`vector_table`: is a section that stores the vector (interrupt) table.
`.ARM.attriubtes` and `.debug_*` contain metadata that will _not_ be loaded onto the target when flashing the binary.

We can use `cargo-objdump` to dissassemble the binary:

```
cargo objdump --bin app --release -- --disassemble --no-show-raw-insn --print-imm-hex
```

From the `cargo-objdump` help page:

```
The arguments specified *after* the `--` will be passed to the proxied tool invocation.

To see all the flags the proxied tool accepts run `cargo-objdump -- -help`.
```

We can see the the options llvm-objdump accepts:

```
--disassemble           Display assembler mnemonics for the machine instructions
--no-show-raw-insn      Do not show raw instruction bytes
--print-imm-hex         Use hex format for immediate values
```

### Running
We can build a hello world example with `cargo build --example hello`.

To run the example, we can invoke `qemu`:

```
qemu-system-arm \
  -cpu cortex-m3 \
  -machine lm3s6965evb \
  -nographic \
  -semihosting-config enable=on,target=native \
  -kernel target/thumbv7m-none-eabi/debug/examples/hello
```

The `-cpu` flag specifies the target CPU model, which will make `QEMU` report certain miscompilation errors to us.

The `-machine` option tells QEMU to emulate a specific board.

`-nographic` avoids launching the `QEMU` GUI.

`-semihosting-config` allows the emulated device to use the host's `stdout`, `stderr` and `stdin`, and to create files on the host.

`-kernel $file` specifies which binary to load an run on the emulated machine.

#### Using a custom runner

See `chapter_2/app/.cargo/config.toml` line 3 for an example of how to override the behavior of cargo run for a specific compilation target

### Debugging

Since the program will be running on a remote device, we will have to perform remote debugging which relies on a client/server paradigm. In this case, our client will be a `GDB` process on our laptop and our server will be the process that is running our embedded program on `QEMU`.

We can launch `QEMU` in debugging mode as follows:

```
qemu-system-arm \
  -cpu cortex-m3 \
  -machine lm3s6965evb \
  -nographic \
  -semihosting-config enable=on,target=native \
  -gdb tcp::3333 \
  -S \
  -kernel target/thumbv7m-none-eabi/debug/examples/hello
```

note the options we added since the last time we compiled the hello example:

`-gdb` tells `QEMU` to wait for a GDB connection on TCP port 3333.
`-s`  tells `QEMU` to freeze the program at startup.

We then invoke gdb with the debug symbols of the example:

```
gdb-multiarch -q target/thumbv7m-none-eabi/debug/examples/hello
```

If we then call `target remote :3333`, gdb will bind to port 3333 to connect to QEMU. We can then debug the program as we usually would with `gdb`

`list` can be used to show source code for a function, e.g. `list main`.

`break $line` can be used to insert a breakpoint directly before the $line.

`continue` will tell `gdb` to run the program until the next breakpoint.

`next` will tell `gdb` to move forward one command.

`step` will execute the next instruction.