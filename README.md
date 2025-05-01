# STM32 MPU6050 Sensor Integration

> This project demonstrates the integration of an MPU6050 accelerometer and gyroscope sensor with the STM32 microcontroller using I2C communication. The sensor reads accelerometer and gyroscope data, processes it, and sends it over UART to a connected PC for further processing or visualization.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Hardware Requirements](#hardware-requirements)
3. [Software Requirements](#software-requirements)
4. [Setup and Configuration](#setup-and-configuration)
5. [Code Explanation](#code-explanation)
6. [Licensing](#licensing)

## Project Overview

The purpose of this project is to interface the MPU6050 sensor with the STM32 microcontroller and collect real-time data for both the accelerometer and the gyroscope. The data is then transmitted via UART to a connected PC at a 10Hz update rate. The sensor reads acceleration values in three axes (X, Y, Z) and rotational speeds (gyroscope values) in three axes (X, Y, Z), which are sent in a binary data packet for efficient transmission.

### Sequence Diagram
![XP5DImCn48Rl-HLpabs8qDQFGm_I9OkwXwLXs-B7fOGubi1kKfDffVvwazKYhegtPFAyppkGMI3bQTUrJ6bo7XRB-cnoVJuUWmeV5uQM31sWbglBqSKepUFnE9KY3QHWh8VXz2gzJg6oXengpHj2TdSxNnVrkk8WtIjwg9LL6-oYWKEfI46Z27CMSodUUwRGODWw6sAynBtH25LvpCy](https://github.com/user-attachments/assets/b0afa724-da73-47c9-8bd4-615c2001b900)

### Functionnality Diagram
![RP2x3i8m34NtV8L7690ehu41WO9u34XLosEsWa4HRL8b3bBvUXeAIyLcttjiwthf6Hs7iX2es3h8ZuVnQ3D94a3VDe8CQwxXa9vcm-amwatvKrCPXLGAyO5Xi8Zk7AGrDtqFZc0_ad1hDajRbi5eXQojwT0NV1242d8othgEcHF36XaXGoRe9O_Tgzz1Ci8hh9IYWfmjYSbz9lvHszf](https://github.com/user-attachments/assets/8de96b32-8d5c-4cc8-ae65-40b5a8da23e0)

### Component Diagram
![TP1D2u9054NtyoiUxWMrZqMBC2Q25iQGkWeN3nrIKaUSCOBelpSZagIrFTnxpvlnfS9MTLL96anlUELWXL6my0vBruLJbvPSMF0a09SH1pHEmA2ZHu7njcXotcAP61Jlpj4NUv5nEM3nsbav4F9QYdRO7Q1UrD6fnRQtCmriW8ggwkeYHSKbt0i47VdTdTwZozgUISXGOLyr2J9UySU](https://github.com/user-attachments/assets/20cca415-b94a-4327-a700-eaae226fd20b)



### Key Features:
- Initialize the MPU6050 sensor.
- Read raw accelerometer and gyroscope data.
- Convert raw data into real-world values (acceleration in g and angular velocity in dps).
- Send the data via UART in a structured packet format (with a checksum).

## Hardware Requirements

- [x] **STM32 Microcontroller**: STM32F4 (or similar series)
- [x] **MPU6050 Accelerometer and Gyroscope Sensor**
- [x] **UART Communication Module**: To send data to a connected PC (e.g., USB to UART converter)
- [x] **Wires and Breadboard**: For connecting the STM32 to the MPU6050

> **PC**: To receive and visualize the data

## Software Requirements

- [x] **STM32CubeMX**: For peripheral initialization (UART, I2C)
- [x] **STM32 HAL Drivers**: For handling hardware abstraction
- [x] **STM32CubeIDE**: To compile and flash the firmware onto the STM32
- [x] **Serial Monitor**: To visualize the UART data (e.g., PuTTY, Tera Term)

## Setup and Configuration

### Hardware Setup:
- Connect the MPU6050 to the STM32 using I2C:
  - **SCL → I2C1_SCL**
  - **SDA → I2C1_SDA**
  - **VCC → 3.3V**
  - **GND → GND**

### Software Setup:
1. Open **STM32CubeMX** and configure the following peripherals:
   - **I2C1** for communication with the MPU6050.
   - **USART2** for UART communication.
   - System Clock Configuration for proper timing.
2. Generate the initialization code and open it in **STM32CubeIDE**.
3. Implement the functionality in `main.c` as described in the code.

### Compiling and Flashing:
- Build the project in **STM32CubeIDE**.
- Flash the firmware to the STM32 using an **ST-Link programmer**.
- Open a serial monitor on your PC (115200 baud rate) to view the transmitted data.

## Code Explanation

### Initialization of MPU6050

In `MPU6050_Init()`, the code interacts with the sensor's registers:
- **WHO_AM_I** register is read to verify the device ID (should return `0x68`).
- Power management is configured to wake up the sensor.
- Data rate is set to **1 kHz** by writing to the `SMPLRT_DIV` register.
- Gyroscope and accelerometer configurations are set to their default ranges (±250 degrees/second for gyro, ±2g for accelerometer).

### Reading Accelerometer and Gyroscope Data

- `MPU6050_Read_Accel()` reads **6 bytes** of data from the accelerometer registers.
- `MPU6050_Read_Gyro()` reads **6 bytes** of data from the gyroscope registers.
- Both functions convert the raw **16-bit** data into human-readable values:
  - Accelerometer values are divided by **16384** (sensor scale for ±2g range).
  - Gyroscope values are divided by **131** (sensor scale for ±250 degrees/second).

### Data Packet and UART Communication

Data is sent as a structured binary packet:
- A **header** (`0xAA`, `0x55`) is added at the beginning of the packet.
- **Accelerometer** data (X, Y, Z) and **Gyroscope** data (X, Y, Z) are added.
- A **checksum byte** is calculated using XOR on all packet bytes for error checking.

The final packet is transmitted via UART at a **10 Hz** update rate.

#### Example Data Packet

| Byte | Description                                  |
|------|----------------------------------------------|
| 0    | Packet Header (0xAA)                        |
| 1    | Packet Header (0x55)                        |
| 2-3  | Accelerometer X (high byte, low byte)       |
| 4-5  | Accelerometer Y (high byte, low byte)       |
| 6-7  | Accelerometer Z (high byte, low byte)       |
| 8-9  | Gyroscope X (high byte, low byte)           |
| 10-11| Gyroscope Y (high byte, low byte)           |
| 12-13| Gyroscope Z (high byte, low byte)           |
| 14   | Checksum byte                               |

## Licensing

This software is licensed under the terms found in the `LICENSE` file. If no license file is included, the software is provided as-is without any warranties.

Feel free to modify and use the code as needed for your projects!
