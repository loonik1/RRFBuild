Introduction
============
This document provides a brief overview of how the STM32F4 version of RRF makes use of the
MCU hardware.


Timers
======
* TIM1 : Software UART (16bit Baud rate * 4 (overampling))
* TIM2 : Hardware PWM
* TIM3 : Hardware PWM
* TIM4 : Hardware PWM
* TIM5 : Step Timer (32 bit 1MHz)
* TIM6 : Unused
* TIM7 : Software PWM (16 bit 1MHz)
* TIM8 : Hardware PWM
* TIM9 : Hardware PWM
* TIM10 : Hardware PWM
* TIM11 : Hardware PWM
* TIM12 : Hardware PWM
* TIM13 : Hardware PWM
* TIM14 : Hardware PWM
* WWDG : Watchdog

PWM outputs
===========
* Hardware PWM is provided using the STM32F4 timers
* Each timer can drive up to 4 GPIO pins, all pins sharing a timer must use the same base frequency
* RRF supports up to 16 software driven PWM channels, these may be used with any GPIO pin

The current allocation of pins to timers is:
* TIM2: PA_0, PA_1, PA_2, PA_3, PA_15, PB_3, PB_10, PB_11
* TIM3: PB_0, PB_1, PB_4, PB_5, PC_7
* TIM4: PB_6, PB_7, PD_12, PD_14, PD_15
* TIM8: PA_5, PB_15, PC_6, PC_8, PC_9
* TIM9: PE_5, PE_6
* TIM10: PB_8, PF_6
* TIM11: PB_9, PF_7
* TIM12: PB_14 PH_6
* TIM13: PA_6, PF_8
* TIM14: PA_7, PF_9

SPI
===
* SPI1 : Shared SPI, Master, main usage SD card interface, does not use DMA
* SPI2 : Slave, SBC or WiFi interface, uses DMA (DMA1_Stream3, DMA1_Stream4)
* SPI3 : Shared SPI, Master, main usage TMC SPI interface
* SWSSP0 : Software SPI, Shared, main usage Thermocouples etc.
* SWSSP1 : Software SPI, Shared, main usage LCD display
* SWSSP2 : Software SPI, shared, main usage TMC SPI interface

SDIO
====
* SDIO : SD card interface on some boards, uses DMA (DMA2_Stream3, DMA2_Stream6)

ADC
===
* ADC1 : 12Bit Analog in + internal ref + mcu temp, uses DMA (DMA2_Stream4)
* ADC2 : unused
* ADC3 : 12Bit Analog in, uses DMA (DMA2_Stream0)

DMA
===
* DMA1_Stream0 : SPI3 RX
* DMA1_Stream1 : Unused
* DMA1_Stream2 : Unused
* DMA1_Stream3 : SPI2 RX
* DMA1_Stream4 : SPI2 TX
* DMA1_Stream5 : SPI3 TX
* DMA1_Stream6 : unused
* DMA1_Stream7 : unused
* DMA2_Stream0 : ADC3
* DMA2_Stream1 : unused
* DMA2_Stream2 : unused
* DMA2_Stream3 : SDIO RX
* DMA2_Stream4 : ADC1
* DMA2_Stream5 : TIM1/GPIO (Soft UART)
* DMA2_Stream6 : SDIO TX
* DMA2_Stream7 : unused

CRC Unit
========
* CRC32 : Used for file I/O and SBC buffers

Flash Memory
============
* 0x8000000 : 32K Bootloader (provided by board)
* 0x8008000 : 16K Interrupt vectors and startup code (bootloader starts loading here)
* 0x800C000 : 16K Reset data (holds diagnostics from previous board resets)
* 0x8010000 : 1024K - 64K Main firmware start

RAM
===
* 0x20000000 : 128K General purpose RAM
* 0x10000000 : 64K CCMRAM unused

SD Card access and RRF configuration
====================================
To enable RRF to operate on a number of different boards it uses a file (board.txt) to specify how the
hardware is configured. This file along with a built in confguration for each board maps the MCU pins to
heaters, drivers etc. However this raises the question of how does RRF read the SD card?

RRF currently supports both SPI and SDIO access to SD cards. There are 3 SPI devices each of which can be
configured to use a selection of I/O pins. Different boards currently use different configurations. Prior
to V3.3 we actively probed the hardware to determine which configuration was in use. However this is risky
as the probing could turn on heaters or other hardware. From V3.3 onwards we make use of the following steps
to determine how to access the SD card:
1. We use a hash of the Bootloader area to identify known boards.
2. If no known board is discovered, we examine the current hardware configuration. The bootloader will often
leave the hardware configured for SD card access.
3. If the above fails we issue an error message and await user permission to actively probe the hardware.