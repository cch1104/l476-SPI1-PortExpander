# l476-SPI1-PortExpander
# STM32L476RG + MCP23S17 SPI GPIO Expander LED Control

## Overview

This project demonstrates SPI communication between an STM32L476RG microcontroller and the MCP23S17 16-bit GPIO Expander.

The STM32 operates as an SPI Master and communicates with the MCP23S17 through SPI2. The MCP23S17 is configured using register-level commands to control an external LED connected to GPIOA0.

The project verifies successful SPI communication by periodically toggling the LED through the MCP23S17 GPIOA register.

---

## Features

* SPI Master communication using STM32 HAL
* MCP23S17 GPIO Expander control
* Register-level device configuration
* Manual Chip Select (CS) management
* LED blinking demonstration
* Embedded firmware development using C
* STM32CubeIDE project

---

## Hardware Components

### Microcontroller

* STM32L476RG

### Peripheral Device

* MCP23S17 16-bit SPI GPIO Expander

### Additional Components

* LED
* 220Ω Resistor
* Breadboard and Jumper Wires

---

## Hardware Connections

| STM32L476RG | MCP23S17 |
| ----------- | -------- |
| SPI2_SCK    | SCK      |
| SPI2_MOSI   | SI       |
| SPI2_MISO   | SO       |
| PC0         | CS       |
| 3.3V        | VDD      |
| GND         | VSS      |

### LED Connection

```text
MCP23S17 GPA0 ---- Resistor ---- LED ---- GND
```

---

## SPI Configuration

The STM32 is configured as SPI Master using SPI2.

### SPI Parameters

| Parameter      | Value            |
| -------------- | ---------------- |
| Mode           | Master           |
| Data Size      | 8-bit            |
| Clock Polarity | Low (CPOL=0)     |
| Clock Phase    | 1 Edge (CPHA=0)  |
| First Bit      | MSB              |
| NSS            | Software Control |

This configuration corresponds to SPI Mode 0, which is supported by the MCP23S17.

---

## MCP23S17 Register Configuration

### Register Definitions

```c
#define IODIRA     0x00
#define MCP_GPIOA  0x12
```

### Register Description

| Register | Address | Description              |
| -------- | ------- | ------------------------ |
| IODIRA   | 0x00    | GPIOA Direction Register |
| GPIOA    | 0x12    | GPIOA Data Register      |

---

## Device Address

The MCP23S17 SPI write opcode is configured as:

```c
char SPI_ADDRESS = 0x40;
```

Address format:

```text
0100 A2 A1 A0 R/W
```

When A2, A1, and A0 are connected to GND:

```text
Write Opcode = 0x40
Read Opcode  = 0x41
```

---

## SPI Write Function

The following function sends data to a specified MCP23S17 register:

```c
void SEND(char RegAddr, char data)
{
    uint8_t buff[3];

    HAL_GPIO_WritePin(GPIOC, CS, GPIO_PIN_RESET);

    buff[0] = SPI_ADDRESS;
    buff[1] = RegAddr;
    buff[2] = data;

    HAL_SPI_Transmit(&hspi2, buff, 3, 1000);

    HAL_GPIO_WritePin(GPIOC, CS, GPIO_PIN_SET);
}
```

SPI Frame Structure:

```text
CS Low
│
├── Opcode (0x40)
├── Register Address
└── Data
│
CS High
```

Example:

```c
SEND(MCP_GPIOA, 0x01);
```

Transmitted Data:

```text
0x40 0x12 0x01
```

Meaning:

```text
Write GPIOA Register
Set GPA0 High
```

---

## GPIO Initialization

GPIOA0 is configured as an output:

```c
SEND(IODIRA, 0xFE);
```

Binary representation:

```text
11111110
```

| Pin       | Direction |
| --------- | --------- |
| GPA0      | Output    |
| GPA1-GPA7 | Input     |

Only GPA0 is used for LED control.

---

## Main Program Operation

The main loop continuously toggles GPIOA0:

```c
while (1)
{
    SEND(MCP_GPIOA, 0);
    HAL_Delay(1000);

    SEND(MCP_GPIOA, 1);
    HAL_Delay(1000);
}
```

Operation:

1. GPA0 set LOW
2. Wait 1 second
3. GPA0 set HIGH
4. Wait 1 second
5. Repeat indefinitely

---

## Expected Result

After programming the STM32:

* SPI2 initializes successfully
* MCP23S17 GPIOA0 is configured as output
* LED connected to GPA0 blinks every second
* SPI communication between STM32 and MCP23S17 is verified

---

## Development Environment

* STM32CubeIDE
* STM32 HAL Driver Library
* Embedded C
* STM32L476RG Nucleo Board

---

## Learning Outcomes

Through this project, I gained hands-on experience with:

* STM32 HAL SPI driver development
* SPI protocol implementation
* MCP23S17 register-level programming
* External peripheral integration
* Embedded firmware development
* Hardware-software verification
* GPIO expander configuration
* Embedded system debugging and troubleshooting

---

