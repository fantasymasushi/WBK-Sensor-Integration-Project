# WBK-Sensor-Integration-Project
Festo Sensor Integration Project for Ponticon System


Ponticon Flow Monitoring System – Code Overview

This repository contains Python scripts for a distributed 4–20 mA flow monitoring system based on:

Raspberry Pi 4 (sensor node)

Raspberry Pi 4 (display node)

M5Stack AIN4-20mA I2C module

Festo SFAH flow sensors

Bluetooth RFCOMM communication

The system reads two flow sensors, converts their signals to physical units (L/min), and visualizes live data on a GUI.

Architecture Overview
Festo SFAH Sensors
        ↓ (4–20 mA)
M5Stack AIN4-20mA (I2C)
        ↓
Raspberry Pi Zero 2 W
  - Reads currents
  - Converts to flow
  - Sends JSON via Bluetooth
        ↓
Bluetooth RFCOMM
        ↓
Raspberry Pi 4
  - Receives JSON
  - Displays flows + delta in GUI

Core Driver
module_4_20ma.py

Low-level I2C driver for the M5Stack AIN4-20mA module 

module_4_20ma

This file is the foundation for all other scripts.

Responsibilities:

Communicates with the module via I2C (smbus2)

Uses a write-register-then-read pattern for reliability

Reads:

12-bit ADC raw values

Raw current registers

Firmware version

Supports up to 4 channels

Handles calibration registers if needed

Why it exists:
The module does not behave reliably with standard SMBus reads. This driver mirrors what works with i2ctransfer on Linux.

Pi Zero 2 W – Sensor Node
zero2w_flow_client.py

Main production client running on the Pi Zero 2 W 

zero2w_flow_client

What it does:

Reads two 4–20 mA channels from the M5Stack module

Converts current → flow rate (L/min)

Computes delta flow

Sends data as JSON lines over Bluetooth RFCOMM to the Pi 4

Automatically reconnects if Bluetooth drops

Data sent (example):

{
  "flow1_lpm": 42.3,
  "flow2_lpm": 45.8,
  "delta_lpm": 3.5,
  "current_ch0_mA": 8.12,
  "current_ch1_mA": 8.85,
  "raw_ch0": 812,
  "raw_ch1": 885
}


Key features:

Robust reconnect loop (handles BlueZ errors like EBUSY)

Fixed send rate (default: 5 Hz)

Designed to run unattended

zero2w_flow_local.py

Local debugging version (no Bluetooth) 

flow_local

What it does:

Reads the same two sensors

Converts current → flow

Prints values directly to the terminal

Use case:

Verifying wiring

Checking sensor behavior

Debugging calibration before enabling Bluetooth

Pi 4 – Display Node
pi4_flow_display_gui.py

Tkinter GUI for live flow visualization 

pi4_flow_display_gui

What it does:

Runs a Bluetooth RFCOMM server

Receives JSON lines from the Zero 2 W

Displays:

Ponticon Flow (CH0)

Source Flow (CH1)

Delta Flow (CH1 − CH0)

Color-codes delta:

Green: small deviation

Yellow: moderate deviation

Red: large deviation

Shows -.-- when disconnected or data is stale

Design goals:

No GUI freeze (Bluetooth runs in a background thread)

Automatic recovery on disconnect

High-contrast display for shop-floor use

Diagnostics & Validation Scripts
read_adc_and_current.py

ADC vs current cross-check tool 

read_adc_and_current

Reads both:

Raw ADC voltage

Raw current register

Purpose:

Verify firmware scaling

Validate that raw / 100 = mA is correct

Debug sensor and module behavior

adc_resolution_detector.py

Detects effective ADC resolution 

adc_resolution_detector

Samples ADC values repeatedly and reports:

Observed min/max range

Whether the ADC behaves like:

12-bit (0–4095)

16-bit scaled

Purpose:

Firmware reverse-engineering

Confidence in ADC interpretation

check_module.py

Minimal health-check script 

check_module

Pings the module

Reads firmware version

Confirms I2C communication

Useful for quick sanity checks.

Read Raw Values.txt

Legacy raw-value reader (early testing) 

Read Raw Values

Continuously prints raw current registers for two channels.
Kept mainly for reference and early bring-up history.