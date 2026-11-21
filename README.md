# Schmetterling Project
Looping Louie on steroids, the game upgrade project

## Overview
A 3-phase motor drives the game. Up to 8 players each have a **User Module**
with two buttons, UP and DOWN, that raise or lower that player's speed
request. The **Main Controller** (Schmetterling_MB, STM32F103C8) polls
all modules round robin over I2C. It adds up the requests of all healthy
modules and runs the motor at the resulting target speed.

```
User Module 0..7  --I2C 400 kHz-->  Main Controller  --3-phase PWM-->  Motor
(2 buttons, health)                 (sum, ramp, ctrl)
```

- **Main Controller HW:** 24 V input, 3 half-bridges (IRS2103 + N-MOSFETs),
  per-phase current sensing, USB-C debug UART, I2C bus to the User Modules
  through a level shifter.
- **I2C:** module address `0x20 + ID` (0x20..0x27). Each poll reads 1 byte:
  `HLTH | ALIVE | PAR | SPEED_REQ[4:0]`. A module with health = 0, a frozen
  ALIVE bit, a parity error or a missing ACK does not count.
- **Scheduling:** 1 ms system tick. Each tick polls one module (~51 us at
  400 kHz), so all 8 modules are polled every 8 ms.

## Folder structure
```
Schmetterling_project/
├── HW/                             Hardware
│   ├── hw_arch_description.txt     Main Controller board functional description
│   └── schmetterling_mb/           KiCad project of the Main Controller board
├── SW/                             Software
│   ├── sw_arch_description.txt     SW architecture, I2C protocol, scheduling
│   ├── MainController/             Firmware for the Main Controller (STM32F103C8)
│   └── UserModule/                 Firmware for the User Module (I2C slave)
├── LICENSE                         MIT-beer extended license
└── README.md
```

## Documentation
- [HW/hw_arch_description.txt](HW/hw_arch_description.txt): power supply,
  power stage, sensing, MCU pinout, interfaces
- [SW/sw_arch_description.txt](SW/sw_arch_description.txt): SW layers, I2C
  addressing and frame, request validation, speed calculation, 1 ms scheduler

## License
MIT-beer extended, see [LICENSE](LICENSE).
