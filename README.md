# Lab6
# STM32 Lab 6 — ADC, DMA and FreeRTOS

## Overview

This repository contains the implementations of Lab 6 using the **STM32 B-L475E-IOT01A2 Discovery Board**.

The lab focuses on Analog-to-Digital Conversion (ADC), timer-triggered sampling, Direct Memory Access (DMA), and task synchronization using FreeRTOS.

Two problems are implemented:

- **Problem 1:** ADC sampling using interrupts and FreeRTOS semaphore synchronization.
- **Problem 2:** ADC sampling using DMA circular mode and FreeRTOS event flags.

## Hardware and Software

**Hardware**
- STM32 B-L475E-IOT01A2 Discovery Board
- USB connection for programming and serial communication

**Software**
- STM32CubeIDE
- STM32CubeMX
- FreeRTOS (CMSIS-RTOS2)
- PuTTY (UART serial terminal)

**UART Configuration**
- Baud rate: 115200
- Data bits: 8
- Stop bits: 1
- Parity: None
- Flow control: None

## Problem 1 — ADC with Interrupts

Directory: `lab6_1/`

### Implementation

The internal temperature sensor of the STM32 is sampled using ADC1.

TIM1 generates a periodic trigger at approximately **10 Hz**, initiating an ADC conversion.

After each conversion, the ADC interrupt callback releases a FreeRTOS semaphore. The ADC task then reads the conversion result, calculates the temperature, and transmits the values through UART.

### Workflow

1. TIM1 generates a periodic trigger.
2. ADC1 performs a temperature sensor conversion.
3. The ADC conversion-complete interrupt occurs.
4. The interrupt callback releases a semaphore.
5. The FreeRTOS task reads and processes the ADC value.
6. The ADC value and calculated temperature are displayed through UART.

### Example Output

```text
ADC Value: 943, Temperature: 28 C
ADC Value: 944, Temperature: 28 C
ADC Value: 943, Temperature: 28 C
```

## Problem 2 — ADC with DMA

Directory: `lab6_2/`

### Implementation

This problem replaces the interrupt-based sample transfer with **DMA in circular mode**.

ADC1 continues to sample the internal temperature sensor using TIM1 as the external trigger source.

A 256-element buffer is used to store the ADC samples.

The DMA controller generates two events:

- **Half Transfer Complete:** The first 128 samples are ready.
- **Transfer Complete:** The remaining 128 samples are ready.

FreeRTOS event flags are used to notify the ADC task when either half of the buffer is ready for processing.

The task then converts each ADC reading into a temperature value and outputs the results through UART.

### Workflow

1. TIM1 periodically triggers ADC1.
2. DMA transfers ADC results into a circular buffer.
3. DMA Half Transfer Complete signals the first buffer half.
4. DMA Transfer Complete signals the second buffer half.
5. FreeRTOS event flags wake the ADC task.
6. The task processes and displays the corresponding samples.
7. DMA continues writing to the buffer automatically.

### Example Output

```text
ADC[0]: 943, Temp: 28 C
ADC[1]: 944, Temp: 28 C
...
ADC[127]: 943, Temp: 28 C
ADC[128]: 944, Temp: 28 C
...
ADC[255]: 943, Temp: 28 C
```

## Project Structure

```text
STM32-Lab6/
├── lab6/
│   └── ...
├── lab6_dma/
│   └── ...
├── .gitignore
└── README.md
```

## How to Run

1. Open STM32CubeIDE.
2. Import the desired project into the workspace.
3. Build the project.
4. Connect the STM32 B-L475E-IOT01A2 board via USB.
5. Flash the firmware onto the board.
6. Open PuTTY and configure the serial connection at 115200 baud.
7. Observe the ADC readings and temperature values.

## Key Concepts

- ADC conversion and internal temperature sensing
- Timer-triggered periodic sampling
- ADC interrupt handling
- DMA circular buffer operation
- DMA half-transfer and transfer-complete callbacks
- FreeRTOS semaphore and event flag synchronization
- UART serial communication

## Notes

The temperature is calculated using the STM32 HAL temperature conversion macro, assuming an analog supply voltage of 3.3 V.

Temperature values may vary depending on the actual supply voltage and sensor calibration.

## Development Environment

- Board: ST B-L475E-IOT01A2
- MCU: STM32L475VGTx
- IDE: STM32CubeIDE 2.2.0
- RTOS: FreeRTOS with CMSIS-RTOS2
