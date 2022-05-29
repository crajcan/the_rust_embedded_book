## Table of Contents<!-- omit in toc -->
- [1.1 Hardware](#11-hardware)
- [1.2 no_std](#12-no_std)

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
