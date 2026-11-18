# Autonomous Soaring – STM32 Sensor Board

## 1. Project Overview

The objective of this project is to develop an electronic system capable of autonomously navigating a model glider from point A to point B without motor assistance.

The glider relies entirely on natural updrafts, such as thermal currents, to gain altitude and extend its flight range. An artificial intelligence system is intended to identify potential thermal updrafts by analysing probabilities and terrain characteristics.

The electronic system collects data from multiple sensors, processes the measurements, and stores the acquired information for post-flight analysis. In addition, sensor data is transmitted in real time to a Raspberry Pi Zero 2 for further processing.

The system is designed around an STM32 microcontroller and integrates several sensors to monitor the glider's position, orientation, altitude, airspeed, and environmental conditions.

## 2. System Architecture

The central component of the system is the STM32U575CIT6Q microcontroller, which handles sensor acquisition, data processing, calculations, and communication with external devices.

The system integrates the following main components:

| Component | Function |
|---|---|
| STM32U575CIT6Q | Main microcontroller for data acquisition and processing |
| ICM-20948 IMU | Measures acceleration, angular velocity, and magnetic field |
| BME280 | Measures atmospheric pressure, temperature, and humidity |
| Grove GPS Air530 | Provides geographical position, altitude, and ground speed |
| Airspeed Sensor (4525DO) | Measures airspeed using differential pressure |
| Raspberry Pi Zero 2 | Receives and processes sensor data in real time |
| SD Card | Stores sensor measurements for post-flight analysis |
| Pixhawk Flight Controller | Optional communication and flight control integration |

## 3. Hardware

### 3.1 Microcontroller

The system uses the **STM32U575CIT6Q**, based on the ARM Cortex-M33 architecture.

The microcontroller was selected because of its low power consumption, processing capabilities, and integrated communication interfaces.

Main specifications:

| Parameter | Specification |
|---|---|
| Microcontroller | STM32U575CIT6Q |
| CPU Core | ARM Cortex-M33 |
| Maximum Performance | 240 MHz |
| Flash Memory | 2 MB |
| SRAM | 786 KB |
| Operating Voltage | 3.3 V |
| Communication Interfaces | SPI, UART, I2C |
| Debug Interface | ST-LINK / SWD |

The microcontroller also provides hardware acceleration features, including:

1. FPU (Floating Point Unit)
2. FMAC (Filter Math Accelerator)
3. CORDIC (Coordinate Rotation Digital Computer)

These features support efficient mathematical operations and digital signal processing.

### 3.2 Sensors

The following sensors are integrated into the system:

**IMU – ICM-20948**

The Inertial Measurement Unit combines three 3D MEMS sensors:

1. Accelerometer
2. Gyroscope
3. Magnetometer

The IMU provides information about the aircraft's orientation, acceleration, and angular velocity.

Communication interface: SPI.

**Environmental Sensor – BME280**

The BME280 integrates three measurement capabilities:

1. Atmospheric pressure
2. Temperature
3. Relative humidity

The pressure measurements are used to estimate altitude.

Communication interface: SPI.

**GPS – Grove GPS Air530**

The GPS module provides geographical positioning and movement information.

Main measurements:

1. Latitude
2. Longitude
3. Altitude
4. Ground speed

Communication interface: UART.

**Airspeed Sensor – 4525DO**

The airspeed sensor measures the differential pressure required to calculate the aircraft's airspeed.

Communication interface: I2C.

## 4. Communication Interfaces

Several communication interfaces are implemented to connect the sensors and external devices.

| Interface | Connected Device | Purpose |
|---|---|---|
| SPI | Raspberry Pi Zero 2 | Real-time sensor data transmission |
| SPI | SD Card | Sensor data storage |
| SPI | ICM-20948 | IMU communication |
| SPI | BME280 | Environmental sensor communication |
| UART | GPS Module | GPS data acquisition |
| I2C | Airspeed Sensor | Airspeed measurement |
| I2C | Experimental Interface | Additional sensor integration |
| UART | Pixhawk | Optional MAVLink communication |
| SWD | ST-LINK | Programming and debugging |

### 4.1 Raspberry Pi Communication

Sensor measurements are transmitted periodically to the Raspberry Pi Zero 2 through an SPI interface.

The transmission frequency is configurable at compile time, with an interval between 100 ms and 2 seconds.

The SPI interface uses the following configuration:

| Parameter | Value |
|---|---|
| Voltage | 3.3 V |
| Protocol | SPI |
| Maximum Frequency | 5 MHz |
| Data Format | 8 bits |
| Clock Phase | Negative edge |
| Slave Select | Active Low |

The transmitted data packet contains 352 bits of measurement information, including:

1. UTC timestamp
2. Aircraft energy variation
3. Orientation (Quaternion components)
4. Longitude
5. Latitude
6. Altitude
7. Ground speed
8. Airspeed

### 4.2 SD Card Storage

Sensor measurements are also stored on an SD card for post-flight analysis.

The stored measurements are intended to contain the same information transmitted to the Raspberry Pi Zero 2.

### 4.3 Pixhawk Integration (Optional)

An additional UART interface is provided for communication with a Pixhawk flight controller.

The MAVLink protocol can optionally be implemented to exchange flight information and support future autonomous flight control functionality.

## 5. Firmware Architecture

The firmware is developed using **STM32CubeIDE** and structured around **FreeRTOS**.

The software architecture is divided into four main task categories.

### 5.1 Main Task

The main task initializes the microcontroller, configures the sensors, and starts the remaining FreeRTOS tasks.

### 5.2 Sensor Tasks

Each sensor has its own dedicated task.

These tasks periodically acquire measurements, convert the raw sensor values into appropriate engineering units, and forward the processed measurements to the Processing Task.

### 5.3 Processing Task

The Processing Task receives measurement data from the individual sensor tasks.

The main responsibilities are:

1. Filtering sensor measurements using FIR filters.
2. Processing sensor data.
3. Calculating relevant flight parameters.
4. Preparing measurement data for transmission.

A Kalman filter for sensor fusion is considered an optional future extension.

### 5.4 Communication Tasks

The Communication Tasks receive processed data and handle communication with external devices.

Their responsibilities include:

1. Periodically transmitting measurement data to the Raspberry Pi Zero 2 via SPI.
2. Storing sensor measurements on the SD card.
3. Optionally implementing MAVLink communication with the Pixhawk flight controller.

## 6. Data Processing and Calculations

The firmware processes the acquired sensor measurements to calculate important flight parameters.

| Parameter | Description | Data Source |
|---|---|---|
| Altitude | Height relative to a reference point | GPS and Barometer |
| Orientation | Pitch, Roll, and Yaw | IMU |
| Airspeed | Speed relative to the surrounding air | Airspeed Sensor |
| Ground Speed | Speed relative to the ground | GPS |
| Energy Variation | Change in the aircraft's total energy | Altitude and velocity |

### 6.1 Altitude Calculation

Altitude is calculated using a weighted average of the GPS altitude and the altitude derived from atmospheric pressure measurements.

The reference altitude is configured using a push button.

### 6.2 Orientation Calculation

The aircraft's orientation is obtained from the IMU measurements.

The orientation is represented using quaternion components.

### 6.3 Airspeed Calculation

Airspeed is calculated from the differential pressure measured by the airspeed sensor.

### 6.4 Ground Speed Calculation

Ground speed is obtained from the GPS module.

### 6.5 Energy Variation

The aircraft's total mechanical energy consists of kinetic and gravitational potential energy:

\[
E_{\text{total}} = E_{\text{kin}} + E_{\text{pot}}
\]

\[
E_{\text{total}} = \frac{1}{2}mv^2 + mgh
\]

Where:

| Variable | Description |
|---|---|
| \(m\) | Aircraft mass |
| \(v\) | Aircraft velocity |
| \(g\) | Gravitational acceleration |
| \(h\) | Altitude |

The energy variation is calculated by comparing the current total energy with the previously calculated total energy.

This information can be used to estimate whether the glider is gaining or losing energy during flight.

## 7. Power Management

The system is designed with low power consumption in mind.

The PCB uses three supply voltage levels:

| Voltage | Purpose |
|---|---|
| 5 V | Main input voltage |
| 3.3 V | Microcontroller and peripheral sensors |
| 1.8 V | IMU supply voltage |

The required voltage levels are generated using switching power supplies (SMPS).

The estimated maximum system power consumption is approximately 0.5 W.

## 8. Project Objectives

The main firmware requirements are:

1. Acquire measurements from all sensors at least every 2 seconds.
2. Calculate altitude, ground speed, and airspeed.
3. Determine the aircraft's orientation (Pitch, Roll, and Yaw).
4. Calculate the aircraft's energy variation.
5. Apply appropriate filtering to sensor measurements.
6. Transmit measurement data periodically to the Raspberry Pi Zero 2 via SPI.
7. Store sensor measurements on the SD card every 2 seconds.

Additional objectives include:

1. Implementing a Kalman filter for sensor fusion.
2. Estimating true altitude using terrain elevation data and GPS coordinates.
3. Implementing MAVLink communication with the Pixhawk flight controller.
4. Supporting additional sensors through the experimental I2C interface.

## 9. Testing and Validation

The system is intended to undergo several validation procedures.

| Test | Method |
|---|---|
| Temperature | Compare sensor measurements using a climate chamber |
| Position | Validate GPS coordinates at predefined reference points |
| Orientation | Compare IMU measurements with an inclinometer and compass |
| Acceleration | Test the PCB using a centrifuge |
| Speed | Perform measurements during a vehicle test |
| Communication | Compare SPI-transmitted data with SD card recordings |

These tests are intended to verify the accuracy of the sensor measurements and the reliability of the communication interfaces.

## 10. Development Environment

The following development tools and technologies are used:

| Tool / Technology | Purpose |
|---|---|
| STM32CubeIDE | Embedded software development |
| STM32CubeMX | Microcontroller configuration |
| FreeRTOS | Real-time task management |
| C / C++ | Firmware development |
| SPI | High-speed peripheral communication |
| UART | GPS and Pixhawk communication |
| I2C | External sensor communication |
| ST-LINK | Firmware programming and debugging |

## 11. Future Development

Potential future improvements include:

1. Implementation of sensor fusion using a Kalman filter.
2. Integration of terrain elevation maps.
3. Development of AI-based thermal detection.
4. Implementation of MAVLink communication.
5. Integration with the Pixhawk flight controller.
6. Optimization of power consumption and processing performance.
7. Further development of autonomous flight capabilities.

---

## Project Information
**Main Controller:** STM32U575CIT6Q  
**Real-Time Operating System:** FreeRTOS  
**Development Environment:** STM32CubeIDE

The project focuses on developing a low-power embedded sensor platform for autonomous model glider applications, providing reliable sensor acquisition, real-time data processing, and communication with external computing and flight-control systems.
