# Electronic-Speed-Controller-Board-ESC-
Electronic Speed Controller (ESC) using PWM control, developed in C for embedded applications.

<img width="1000" height="841" alt="ESC Perspective View-Photoroom" src="https://github.com/user-attachments/assets/cd50b3fc-694a-4535-87f1-f6b4a4ee8f9f" />
 <p align="center">
 <i>Figure 1. Electronic Speed Controller Perspective View.</i>
</p>


## Project Overview

This project focuses on the design and implementation of a sensorless Brushless DC (BLDC) Electronic Speed Controller (ESC) developed as part of a graduation thesis. The system integrates both hardware and firmware, aiming to achieve reliable motor control using low-cost components and an open-source development approach.

The ESC is built around an ATmega328P microcontroller and operates using Back-EMF zero-crossing detection to determine rotor position without the use of Hall sensors. This sensorless approach reduces system complexity, cost, and mechanical dependency while maintaining efficient motor commutation.

The firmware is written in C/Arduino-based embedded C, directly configuring microcontroller registers for precise timing, PWM generation, and interrupt handling. The ESC accepts a standard RC PWM input signal (1000–2000 µs) and dynamically maps it to motor speed by adjusting PWM duty cycles.

On the hardware side, a custom PCB was designed, including MOSFET driver stages, power regulation, signal conditioning, and protection considerations. The project also includes EEPROM-based parameter storage, allowing the ESC to automatically learn and store throttle range values during configuration.

This project demonstrates practical experience in:

- Embedded systems programming

- Power electronics

- Motor control theory

- PCB design and hardware–software co-design


## Features

- Sensorless BLDC Motor Control
Uses Back-EMF zero-crossing detection for rotor position estimation without Hall sensors.

- Custom ESC Firmware
Firmware developed in embedded C with direct register-level control for timers, PWM, and interrupts.

- Standard RC PWM Input Support
Compatible with 1000–2000 µs PWM signals from RC receivers or flight controllers.

- Automatic Throttle Range Calibration
Throttle minimum and maximum values are detected on startup and stored in EEPROM.

- EEPROM-Based Parameter Storage
Persistent storage of configuration parameters across power cycles.

- Multi-Step Commutation Sequence
Six-step trapezoidal commutation with comparator-based zero-cross detection.

- Audible Status Feedback
Startup, configuration, and error states indicated via buzzer tones at different frequencies.

- Custom PCB Design
Includes schematic design, PCB layout, Gerber files, and Bill of Materials (BOM).

- Low-Cost and Open-Source Approach
Designed using widely available components and open development tools.

## Hardware Design

The hardware design of this Electronic Speed Controller (ESC) is based on a modular and clearly structured three-phase inverter topology, optimized for sensorless BLDC motor control. The system is powered by a Li-Po battery, which serves as the main DC energy source and is capable of supplying the high current required by the motor. To support the low-voltage control electronics, the battery voltage is stepped down using a high-efficiency switching regulator (MP2307DN-LF-Z), providing a stable 5 V supply rail for the microcontroller and auxiliary circuitry. The use of a switching regulator instead of a linear solution significantly improves efficiency and reduces thermal losses, which is critical for battery-powered systems.

<br>
<p align="center">
<img width="854" height="294" alt="Başlıksız DiyagramDD drawio" src="https://github.com/user-attachments/assets/bbcbe7e7-aa70-4cf2-937e-31ce7ce33d58" />
</p>
<p align="center">
  <i>Figure 2. Overall hardware topology of the Electronic Speed Controller.</i>
</p>

<br>

At the core of the control system lies the ATmega328P microcontroller in a 32-pin LQFP package. The microcontroller is responsible for PWM signal generation, RC input signal processing, commutation timing, and Back-EMF zero-crossing detection. One of the key reasons for selecting the ATmega328P is its built-in analog comparator, which enables sensorless motor control without the need for external Hall sensors. This allows rotor position estimation by monitoring the floating motor phase voltage during each commutation step.

The motor drive stage consists of six IRLR7843 N-channel power MOSFETs arranged in a three-phase half-bridge configuration. These MOSFETs were selected due to their low RDS(on), logic-level gate drive compatibility, and high current handling capability, making them suitable for low-voltage, high-current BLDC applications. Each motor phase is driven by a complementary high-side and low-side MOSFET pair, allowing precise control of current flow through the motor windings.

Gate control for the MOSFETs is provided by three IR2101 high-side / low-side gate driver ICs, with each driver dedicated to one motor phase. The IR2101 enables reliable high-side N-channel MOSFET driving through internal level shifting and bootstrap circuitry. This architecture ensures fast switching transitions, proper gate voltage levels, and safe operation by minimizing the risk of shoot-through conditions.

The three motor phases are structured as AH/AL for Phase A, BH/BL for Phase B, and CH/CL for Phase C, where the high-side (H) MOSFET connects the phase to the positive supply rail and the low-side (L) MOSFET connects it to ground. During operation, the ESC follows a six-step trapezoidal commutation sequence. At any given time, one phase is actively driven high, one phase is driven low, and the remaining phase is left electrically floating. The floating phase voltage is used for Back-EMF sensing, allowing the microcontroller to detect zero-crossing events and determine the optimal timing for the next commutation step.


<br>
<p align="center">
<img width="800" height="693" alt="ESC-Circuit" src="https://github.com/user-attachments/assets/ba947ac8-0cbf-447e-9acf-5666cd2943c0" />
</p>
<p align="center">
  <i>Figure 3. ESC Circuit Diagram.</i>
</p>



The complete electrical implementation of this architecture can be seen in the provided schematic diagram, which illustrates the power stage, gate driver connections, microcontroller interfacing, and Back-EMF sensing paths. The schematic highlights the separation between high-power and low-power domains, ensuring both electrical safety and signal integrity. Special attention has been given to grounding, decoupling, and signal routing to improve noise immunity and overall system reliability.

<br>
<p align="center">
<img width="600" height="693" alt="ESC Bottom 2D" src="https://github.com/user-attachments/assets/54609cc3-c70d-4bf7-9333-5572e54b7d15" />
</p>
<p align="center">
  <i>Figure 4. ESC Bottom 2D View.</i>
</p>


Overall, the hardware design achieves a balance between performance, simplicity, and educational clarity. It demonstrates a fully functional sensorless BLDC ESC architecture using widely available components, making it suitable for academic projects, prototyping, and further development.

<br>
<p align="center">
<img width="800" height="627" alt="ESC Bottom View" src="https://github.com/user-attachments/assets/be542494-ea07-47a0-9b2c-e29843bc75d3" />
</p>
<p align="center">
  <i>Figure 5. ESC 3D View.</i>
</p>


## Firmware / Software Design

The ESC firmware is implemented in C/C++ using the Arduino framework and is specifically designed for real-time, sensorless BLDC motor control. The software architecture follows a modular design approach, where motor control logic, hardware abstraction, persistent configuration storage, and user feedback mechanisms are separated into dedicated source and header files. This structure improves maintainability, enhances code readability, and allows individual subsystems to be modified or extended independently without affecting the overall system behavior.

### PWM Input Measurement and Processing

The main firmware file (ESC_V3_Code.ino) handles system initialization, PWM input measurement, motor state control, and commutation sequencing. During startup, all GPIOs, timers, and internal peripherals are configured. The incoming PWM signal from the RC receiver is measured and mapped to a valid motor speed range while enforcing predefined safety limits.

```c
if (PWM_INPUT > PWM_IN_MAX) PWM_INPUT = PWM_IN_MAX;
if (PWM_INPUT < PWM_IN_MIN) PWM_INPUT = PWM_IN_MIN;

```
This ensures that invalid or noisy input signals cannot command unsafe motor behavior.
<br>
<br>
### Sensorless Commutation and Back-EMF Detection

Motor commutation is implemented using a six-step trapezoidal algorithm. At each step, two phases are actively driven while the third phase is left floating. The floating phase voltage is monitored using the ATmega328P’s internal analog comparator to detect Back-EMF zero-crossing events, which are used to determine the correct timing for the next commutation step.

This method enables fully sensorless operation without external position sensors, reducing system complexity and hardware cost.
<br>
<br>
### PWM Generation and Motor Speed Control

Motor speed is controlled by adjusting the PWM duty cycle applied to the active MOSFETs. Hardware timers are used to generate stable PWM signals, allowing precise control over the effective motor voltage.

```c
#define PWM_max_value 255
#define PWM_min_value 35
```
These limits prevent operation in regions where torque is insufficient or switching losses become excessive.
<br>
<br>
### Non-Volatile Configuration Storage (EEPROM)

Calibration parameters such as minimum and maximum PWM input values are stored in EEPROM to persist across power cycles. A generic EEPROM abstraction layer allows arbitrary data types to be stored and retrieved safely.

```c
EEPROM_writeAnything(PWM_IN_MIN_ADRESS, PWM_IN_MIN);
EEPROM_readAnything(PWM_IN_MAX_ADRESS, PWM_IN_MAX);
```
This feature eliminates the need for repeated calibration and improves user experience.
<br>
<br>
### Audible Feedback System

An audible feedback system is implemented using a buzzer to indicate system states such as startup, calibration, and operational modes. Different tones are generated by toggling GPIO pins with precise timing delays.

```c
void beep_1KHZ(int milliseconds);
void beep_2KHZ(int milliseconds);
void beep_3KHZ(int milliseconds);
```
Audible feedback provides a simple yet effective human–machine interface without requiring additional displays or communication protocols.
<br>
<br>
### Global Configuration and State Management

System-wide parameters, operational flags, and timing variables are centralized in a dedicated header file. These variables track motor state, PWM thresholds, timing counters, and safety conditions such as motor stop detection.

```c
bool MOTOR_SPINNING = false;
bool ESC_MODE_ON = false;
bool PWM_RANGE_SET = false;
```
Centralizing these definitions ensures consistent behavior across the firmware and simplifies debugging.
<br>
<br>
### Software Design Summary

The firmware is designed to complement the hardware architecture by leveraging the microcontroller’s internal peripherals to their full potential. By combining modular code organization, sensorless control algorithms, and persistent configuration storage, the software achieves reliable real-time motor control while remaining compact, readable, and suitable for academic and professional embedded systems development.

### Firmware Upload and Programming Methodology

During the software development process, a USBasp programmer and an FTDI (USB-to-UART) interface are utilized to program the ATmega328P microcontroller. The microcontroller used in the ESC design is provided as a raw device without any factory-installed bootloader, which means that direct hardware-level access is required for initial programming.

<br>
<p align="center">
<img width="600" height="686" alt="sss" src="https://github.com/user-attachments/assets/62ff1554-7819-4184-9dfb-b28bead07a07" />
</p>
<p align="center">
  <i>Figure 6. Overall firmware topology of the Electronic Speed Controller.</i>
</p>




In the first stage, the bootloader is written to the microcontroller using the In-System Programming (ISP) method. This low-level programming approach provides direct access to the microcontroller through the SPI lines (MOSI, MISO, SCK, and RESET). A widely supported programmer, USBasp, is employed to establish direct communication between the computer and the microcontroller, enabling reliable bootloader installation.

Once the bootloader has been successfully uploaded via the ISP interface, the system becomes ready for higher-level programming. At this stage, an FTDI module is connected to enable USB-to-serial (UART) communication, allowing firmware to be uploaded through the serial port. This programming sequence is a common and necessary practice when working with microcontrollers that do not include a pre-installed bootloader and represents a critical step in ensuring proper software–hardware integration.

Following bootloader installation, firmware development and code uploads can be performed using development tools such as Arduino IDE, Atmel Studio, or AVRDude. The use of the FTDI interface significantly simplifies the programming workflow, enabling rapid firmware updates, debugging, and functional testing without requiring repeated low-level programming. This approach accelerates the development cycle and supports a flexible, maintainable, and easily updatable software architecture for the ESC system.


## Repository Structure

The repository is organized in a modular and hierarchical manner to clearly separate
firmware and hardware-related files. This structure improves readability, simplifies
navigation, and allows independent development and maintenance of software and hardware
components.

```Repository Structure

ESC_V3_Code/
├── ESC_V3_Code.ino          # Main firmware source file
├── Beeps.h                  # Audible feedback routines
├── EEPROMAnything.h         # EEPROM read/write utilities
├── Variables.h              # Global definitions and system parameters

ESC_V3_Hardware/
├── PCB_Documents/
│ ├── ESC_Schematic.pdf        # Exported circuit schematic (PDF)
│ ├── ESCV3.PcbDoc             # PCB layout file
│ ├── ESCV3.PcbLib             # PCB footprint library
│ ├── ESCV3.PrjPcb             # Altium project file
│ ├── ESCV3.PrjPcbStructure    # Project structure and hierarchy
│ ├── ESCV3.SchDoc             # Schematic design file
│ └── ESCV3.SchLib             # Schematic symbol library
│
├── Gerber_Files/
│   ├── TopCopper.gbr
│   ├── BottomCopper.gbr
│   ├── SolderMask_Top.gbr
│   ├── SolderMask_Bottom.gbr
│   ├── Silkscreen_Top.gbr
│   └── DrillFiles.drl
│
├── Documentation/
│   ├── BOM.xlsx             # Bill of Materials
│   ├── Schematic.pdf        # Exported circuit schematic
│   └── Hardware_Photos/     # Assembly and board images

```

## Getting Started

This section describes the complete process required to assemble, program, and prepare the ESC for operation, starting from hardware assembly to firmware upload and final calibration.

The assembly process begins with soldering all electronic components onto the PCB according to the schematic. Special attention is given to the power input terminals (+ and −), where a thick solder layer is applied to improve current handling capability and enhance thermal dissipation. After reinforcing these pads, power supply cables are soldered to the corresponding terminals to allow stable connection to the main power source.

<br>
<p align="center">
<img width="500" height="686" alt="123" src="https://github.com/user-attachments/assets/b4c1bc06-5666-46ce-9b84-f1607a673a7f" />
</p>
<p align="center">
  <i>Figure 7. Soldering and assembly processes.</i>
</p>


Next, the motor phase connections are prepared. Three high-current output pads connected to the power MOSFET stages are used for the motor phases. Three cables are soldered to these pads, forming the Phase A, Phase B, and Phase C outputs required to drive the BLDC motor.

Following the power stage assembly, the PWM signal wiring is completed. The signal cable from the RC receiver is connected to the designated PWM input pin, along with the required ground reference, enabling throttle control via an external controller.

Once the hardware setup is completed, the software preparation stage begins. Since the ATmega328P microcontroller does not include a factory-installed bootloader, an initial bootloader upload is required. This is performed using a USBasp programmer via the ISP interface. After the bootloader is installed, firmware can be uploaded through the serial interface using an FTDI module. Detailed firmware architecture and programming steps are described in the Firmware section.

After firmware upload, the ESC is ready for operation. The motor phases and power supply are connected, and the system is powered on. Throttle calibration is then performed using a standard RC transmitter, following the same procedure commonly used in commercial ESCs: the throttle stick is first set to the maximum position during power-up, followed by an audible confirmation beep, then moved to the minimum position where a second beep confirms successful calibration. After this process, the ESC is fully calibrated and ready for use in real-world applications.

## Limitations and Future Work

The results obtained in this study demonstrate that a basic Electronic Speed Controller (ESC) can be designed and implemented in a functional and cost-effective manner. However, the current system does not include several advanced ESC features commonly found in commercial or high-performance applications. The following improvements are proposed for future work:

- Closed-Loop Control: The system currently operates in open-loop mode. By integrating Hall sensors or utilizing back-EMF (BEMF) feedback, closed-loop speed control can be implemented, allowing direct motor speed measurement and improved control accuracy.

- Thermal and Current Protection: The present design does not monitor temperature or phase current. Integrating thermal sensors for MOSFET temperature monitoring and current sensors for phase current measurement would significantly enhance system safety and reliability.

- Field-Oriented Control (FOC): The existing control algorithm is based on trapezoidal commutation. Implementing a Field-Oriented Control (FOC) algorithm could improve efficiency, torque smoothness, and overall motor performance.

- High-Frequency PWM Operation: Increasing the PWM switching frequency beyond the audible range could reduce acoustic noise during ESC operation.

- Wireless Communication and Calibration: A UART-based or Bluetooth communication interface could be added to enable wireless parameter tuning, calibration, and real-time monitoring.

- Multi-ESC Synchronization: For applications requiring multiple ESCs operating simultaneously, such as multirotor platforms, synchronization mechanisms could be developed to ensure coordinated operation.

By implementing these enhancements, the system’s safety, control precision, and applicability would be significantly improved, enabling its use in advanced applications such as autonomous aerial vehicles.

## Conclusion

In this study, a functional and cost-effective Electronic Speed Controller (ESC) was successfully designed and implemented by integrating power electronics, embedded firmware, and real-time control principles. The project demonstrates that a reliable sensorless BLDC motor control system can be achieved using a minimal hardware platform and a modular software architecture. Throughout the development process, emphasis was placed on hardware–software integration, safety-oriented design, and practical usability. The resulting ESC provides a solid foundation for further enhancements and serves as a strong demonstration of embedded systems and power electronics engineering skills.

## License

This project is licensed under the MIT License.
You are free to use, modify, and distribute this project for personal or commercial purposes, provided that the original license and copyright notice are included.

## Author

Alperen Ergül
Electrical and Electronics Engineer

Email: alp-ergl@hotmail.com

GitHub: https://github.com/AlperErgl

LinkedIn: https://www.linkedin.com/in/alperen-erg%C3%BCl-428699202/











