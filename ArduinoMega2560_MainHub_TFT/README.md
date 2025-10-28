---
author:
- STEM Club, Bunker Hill Community College
date: 2025-10-28
title: |
  **Arduino Mega 2560 HUB Node**  
  Smart Garden System Documentation
---

# Overview

**Node:** Arduino Mega 2560 + TFT + NRF24L01+  
**Purpose:**

- Receive soil sensor packets via RF.

- Decide pump ON/OFF using hysteresis, timeout, and failsafe.

- Transmit pump commands to the pump node.

- Display live data and logic status on the 3.5” MCUFRIEND TFT.

# Hardware Connections

## TFT (MCUFRIEND 8-bit parallel)

|       |                                          |
|:------|:-----------------------------------------|
| D0–D7 | $`\rightarrow`$ D22–D29                  |
| RS    | $`\rightarrow`$ D38                      |
| WR    | $`\rightarrow`$ D39                      |
| RD    | $`\rightarrow`$ D40                      |
| CS    | $`\rightarrow`$ D41                      |
| RST   | $`\rightarrow`$ D49                      |
| LED   | $`\rightarrow`$ 5V via 100–220Ω resistor |
| VCC   | $`\rightarrow`$ 5V                       |
| GND   | $`\rightarrow`$ GND                      |

## nRF24L01+ with 3.3V Adapter

|      |                                 |
|:-----|:--------------------------------|
| CE   | $`\rightarrow`$ D46             |
| CSN  | $`\rightarrow`$ D47             |
| SCK  | $`\rightarrow`$ D52             |
| MOSI | $`\rightarrow`$ D51             |
| MISO | $`\rightarrow`$ D50             |
| VCC  | $`\rightarrow`$ 5V (to adapter) |
| GND  | $`\rightarrow`$ GND             |

**Decoupling:** 47–100µF electrolytic + 0.1µF ceramic at adapter 3.3V
rail.

## SPI Protection

- Keep SPI master mode: D53 = OUTPUT, HIGH

- Release SD/touch CS lines: D10 = HIGH, D4 = HIGH

# Logic Flow and Operation

- Pump <span style="color: green!50!black">ON</span> if Soil ≤ 40%.

- Pump <span style="color: red!70!black">OFF</span> if Soil ≥ 55% or
  runtime \> 5 min.

- No packets \> 5 s $`\rightarrow`$
  <span style="color: red!80!black">Failsafe OFF</span>.

- Cooldown 1 min after timeout before retry.

## Thresholds and Timers

    ON_THRESH        = 40
    OFF_THRESH       = 55
    NO_SOIL_MS       = 5000
    PUMP_MAX_ON_MS   = 300000
    PUMP_COOLDOWN_MS = 60000

# Code Libraries

- **SPI.h** — Serial Peripheral Interface core.

- **RF24.h** — nRF24L01+ transceiver control.

- **Adafruit_GFX.h** — Base drawing primitives.

- **MCUFRIEND_kbv.h** — TFT shield driver.

# Key Library Functions

## RF24 Library

`begin(), isChipConnected(), setChannel(), setDataRate(), setPALevel(),`  
`setAutoAck(), setRetries(), disableDynamicPayloads(), openReadingPipe(),`  
`openWritingPipe(), startListening(), stopListening(), available(), read(), write()`

## MCUFRIEND_kbv / Adafruit_GFX

`readID(), begin(), setRotation(), fillScreen(), fillRect(), drawRect(), drawLine(),`  
`setTextColor(), setTextSize(), setCursor(), print()`

# Color Coding Scheme

- <span style="color: blue">**Hardware:**</span> Arduino, TFT, RF
  modules

- <span style="color: green!60!black">**Communication:**</span> RF24
  channels, pipes, packet data

- <span style="color: orange!90!black">**Logic/Analytics:**</span>
  Thresholds, timers, decisions

- <span style="color: red!80!black">**Failsafe:**</span> Timeouts, link
  loss, safety overrides

# RF and Logic State Machine

<div class="center">

</div>

# Serial Diagnostics Example

    [MEGA] boot
    [MEGA] TFT ID=0x9486
    [MEGA] RF listening SOIL; chip detected
    [MEGA] SOIL seq=123 raw=512 pct=37 T=24.5C H=41.0%
    [MEGA] TX->PUMP seq=7 pump=ON soil=37 reason=0 -> OK

# Testing Procedure

1.  Power Mega + TFT + radio (with proper decoupling).

2.  Open Serial Monitor @ 115200 baud.

3.  Confirm TFT ID printout and RF chip detection.

4.  Start Soil TX node on pipe `S̈OIL1̈`.

5.  Observe UI updates and logic transitions:

    - Soil ≤ 40% → Pump <span style="color: green!50!black">ON</span>.

    - Soil ≥ 55% or ON \> 5 min → Pump
      <span style="color: red!70!black">OFF</span>.

    - Link loss \> 5 s → <span style="color: red!80!black">Failsafe
      OFF</span>.

# Conclusion

The Arduino Mega 2560 HUB node acts as a stable backbone of the Smart
Garden system, integrating RF communication, intelligent
decision-making, and a graphical interface. Its layered design ensures:

- Robust wireless reception with NRF24L01.

- Safe and efficient control of pump logic.

- Real-time feedback and monitoring via TFT display.

The system can be further extended with Ethernet/cloud logging or
AI-assisted environmental regulation for adaptive irrigation strategies.
