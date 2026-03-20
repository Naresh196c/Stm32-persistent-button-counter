# Persistent Button Press Counter — STM32F446RE

A technician presses a button each time a service is performed. The system counts every press, stores it in internal flash, and recalls it on every reboot — even after a complete power loss. No external EEPROM, no battery-backed RAM. Just the MCU's own flash and a UART terminal.

> Built at AISSMS Institute of Information Technology, Pune  
> Naresh Choudhary Atharva Bobade · Siddharth Dere · 

---

## Scenario

A maintenance team uses an STM32 Nucleo board as a service counter. Each time a technician services a machine, they press the onboard button. The system must:

1. Survive power cuts without losing the count
2. Display the total service count on every startup via UART
3. Reject false presses caused by button bounce
4. Give visual confirmation (LED blink) on every valid press

The counter resets to 0 after 5 presses (configurable), erasing and rewriting the flash sector cleanly.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      STM32F446RE                        │
│                                                         │
│   PC13 ──► GPIO Poll ──► Debounce (50ms)                │
│                               │                         │
│                        Valid Press?                      │
│                          │       │                       │
│                         YES      NO ──► keep polling     │
│                          │                               │
│                    Increment Counter                     │
│                          │                               │
│              ┌───────────┴────────────┐                  │
│              │                        │                  │
│        Blink LED (PA5)     Write to Flash Sector 7       │
│                                 │                        │
│                        0x08060000 (32-bit word)          │
│                                 │                        │
│                        HAL_FLASHEx_Erase()               │
│                        HAL_FLASH_Program()               │
│                                                         │
│   Power ON / Reset                                      │
│        │                                                 │
│   Read_ButtonCount() ◄── Flash Sector 7                  │
│        │                                                 │
│   HAL_UART_Transmit() ──► USART2 (PA2) ──► Terminal     │
└─────────────────────────────────────────────────────────┘
```

**Data flow summary:**  
Button → GPIO → Debounce → Counter → Flash (persist) → UART (report)

---

## Build Steps

**Requirements**
- STM32CubeIDE v1.12.1 or later
- STM32F446RE Nucleo-64 board
- USB cable (for power + virtual COM port)
- Serial terminal: PuTTY / Tera Term / STM32CubeIDE built-in console

**Steps**

```bash
# 1. Clone the repo
git clone https://github.com/Naresh196c/Stm32-persistent-button-counter.git

# 2. Open STM32CubeIDE
#    File → Open Projects from File System → select the cloned folder

# 3. Build
#    Project → Build All   (or Ctrl + B)
#    Confirm: no errors in the Console panel

# 4. Flash to board
#    Run → Debug  (connects, flashes, and halts at main)
#    Then: Run → Resume  (or press F8)
```

No external libraries or package managers needed. Everything uses the STM32 HAL bundled with STM32CubeIDE.

---

## Run Instructions

1. Connect the Nucleo board via USB
2. Open your serial terminal — select the board's virtual COM port
3. Set baud rate to **115200**, 8N1, no flow control
4. Reset the board (black button) or power-cycle it
5. Watch the startup message appear, then press the blue button (B1) to increment

**Button behavior**
- Single press → count goes up by 1, saved to flash immediately
- Rapid pressing → still only increments by 1 (debounce active)
- Count reaches 6 → flash erased, counter resets to 0, message printed
- Reset / power loss → count is read back from flash on next boot

---

## System Calls & HAL Functions — Where and Why

| Function | Where Used | Purpose |
|----------|-----------|---------|
| `HAL_Init()` | `main()` startup | Initialises HAL tick timer, sets up flash interface and SysTick |
| `SystemClock_Config()` | `main()` startup | Configures PLL to run the CPU at 84 MHz from the HSI oscillator |
| `MX_GPIO_Init()` | `main()` startup | Sets PC13 as input (button) and PA5 as push-pull output (LED) |
| `MX_USART2_UART_Init()` | `main()` startup | Configures USART2 at 115200 baud, 8N1, TX+RX on PA2/PA3 |
| `HAL_GPIO_ReadPin()` | Main poll loop | Reads PC13 level to detect button press (active LOW) |
| `HAL_Delay(50)` | After first press detect | Software debounce — waits 50 ms then re-checks pin to confirm press |
| `HAL_GPIO_WritePin()` | After confirmed press | Drives PA5 HIGH then LOW to blink the LED |
| `HAL_FLASH_Unlock()` | `Write_ButtonCount()` | Removes write protection before any flash operation |
| `HAL_FLASHEx_Erase()` | `Write_ButtonCount()` | Erases Sector 7 — required before writing (flash bits can only go 1→0, erase resets to all 1s) |
| `HAL_FLASH_Program()` | `Write_ButtonCount()` | Writes the new 32-bit counter value to address `0x08060000` |
| `HAL_FLASH_Lock()` | `Write_ButtonCount()` | Re-enables write protection after the operation |
| `HAL_UART_Transmit()` | Startup + every press | Sends formatted count string over USART2 to the terminal |
| `Error_Handler()` | Flash erase fail path | Disables interrupts and hangs — signals a fatal flash operation failure |

---

## Sample Run Output

```
System Booted. Button pressed: 3 times

 Button pressed: 4 times

 Button pressed: 5 times

Count exceeded 5.
 Flash erased, count reset.

 Button pressed: 1 times
```

On the next power-up after the session above:

```
System Booted. Button pressed: 1 times
```

The count survived the power cycle — fetched directly from flash Sector 7 at `0x08060000`.

---

*STM32CubeIDE · HAL Library · C · Internal Flash Storage*
