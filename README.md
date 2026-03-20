🚀 Persistent Button Press Counter using STM32 Flash
📌 Project Overview

This project implements a persistent button press counter using the STM32F446RE microcontroller.

Each time a button is pressed:

The counter increments

The value is stored in internal Flash memory

Since Flash is non-volatile, the count remains محفوظ (stored) even after:

Power OFF

System reset

On every startup, the system reads the stored value and displays it using UART, making it useful for maintenance logs and service tracking systems.

🎯 Objectives

Store button press count in Flash memory

Ensure data persists after reset and power loss

Implement accurate button detection with debounce

Display count using UART communication

Provide LED indication on valid button press

Verify system through multiple resets and power cycles

⚙️ Features

✅ Persistent storage using internal Flash

✅ Debounced button input (no false triggering)

✅ UART output for monitoring

✅ LED feedback on button press

✅ Automatic reset when count exceeds limit

✅ Reliable flash read/write operations

🧠 System Working (Simple Explanation)

System starts → reads count from Flash

Displays count on UART

Waits for button press

When pressed:

Debounce applied

LED glows

Count increases

Stored in Flash

If count > 5:

Flash erased

Count reset to 0

🔄 System Flow Diagram
START
  |
  v
Read Count from Flash
  |
  v
Display on UART
  |
  v
Wait for Button Press
  |
  v
Debounce Check
  |
  v
Increment Count
  |
  v
Count > 5 ?
  |        |
 YES       NO
  |         |
Erase Flash  Write Count
Reset Count      |
  |              |
  +-------> Display Count
🔌 Hardware Requirements

STM32F446RE (Nucleo Board)

Push Button (PC13)

LED (PA5)

USB Cable

💻 Software Requirements

STM32CubeIDE

HAL Drivers

Serial Terminal (PuTTY / Tera Term)

📍 Flash Configuration
Parameter	Value
Sector Used	Sector 7
Address	0x08060000
Data Type	32-bit
🔧 Key Functional Modules
🔹 Read_ButtonCount()

Reads value from Flash

If empty (0xFFFFFFFF) → returns 0

🔹 Write_ButtonCount()

Erases Flash sector

Writes updated count

🔹 Erase_Flash_Sector7()

Clears Flash memory safely

🔹 GPIO

PC13 → Button input

PA5 → LED output

🔹 UART (USART2)

Baud Rate: 115200

Used for displaying count

📊 Example Output
System Booted. Button pressed: 0 times

Button pressed: 1 times
Button pressed: 2 times

Count exceeded 5.
Flash erased, count reset.

Button pressed: 1 times
🧪 Test Cases Summary
Test Case	Description	Result
TC1	First startup	Count = 0
TC2	Button press	Count increments
TC3	Rapid press	Single increment
TC4	Reset	Value persists
TC5	Power loss	Value persists
⚠️ Challenges & Solutions
Challenge	Solution
Flash cannot overwrite	Used erase before write
Power loss corruption	Checked for 0xFFFFFFFF
Button noise	Added 50 ms debounce
Flash limited life	Minimized writes
UART overlap	Added delays
🏆 Results

Counter successfully stored in Flash

Data retained after reset and power loss

UART output verified system behavior

Debounce ensured accurate counting

LED provided visual confirmation

🔮 Future Improvements

Interrupt-based button handling

EEPROM emulation

Wear-leveling for Flash life

LCD/OLED display integration

Modular code structure

📚 Learning Outcomes

Flash memory handling in STM32

GPIO interfacing

UART communication

Embedded C with HAL

Data persistence techniques

👨‍💻 Team Members

Atharva Bobade

Siddharth Dere

Naresh Choudhary

🏫 College

AISSMS Institute of Information Technology, Pune

📅 Date

07/11/2025

📖 References

STM32F446RE Datasheet

STM32 HAL Documentation

Flash Programming Manual (PM0075)

STM32CubeIDE Guide

Online Embedded Tutorials
