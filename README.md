Persistent Button Press Counter using STM32 Internal Flash

This project implements a persistent button press counter on the STM32F446RE microcontroller.
Each time a button is pressed, the count is incremented and stored in internal flash memory, ensuring the value is retained even after reset or power loss.

On every system startup, the stored count is read from flash and displayed via UART, making it suitable for maintenance logs and service tracking systems.
