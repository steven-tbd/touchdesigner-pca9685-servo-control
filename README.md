# Scalable Servo Control with TouchDesigner and PCA9685

A control system for driving arrays of hobby servos in paired 2-DOF X/Y configurations, using TouchDesigner and an ESP32 microcontroller.

## Overview

The system sends OSC messages over UDP from TouchDesigner to an ESP32, which routes them through a PCA9685 driver board to control servo pairs. The prototype drives 12 servos across 6 paired mechanisms. The architecture scales to larger arrays by adding driver boards.

Three control modes are available in the TouchDesigner network: live gesture control via Leap Motion, a manual slider UI, and pre-programmed keyframe sequences through the Animation COMP.

![Servo Configuration](servo_configuration_image.png)

## Signal Flow

```
[Leap Motion] ---> (USB HID) ---> [TouchDesigner]
[TouchDesigner] ---> (Wi-Fi / OSC UDP) ---> [ESP32] ---> (I²C) ---> [PCA9685] ---> (PWM) ---> [12x Hobby Servos]
```

**2-DOF pairing.** Servos are organized in X/Y pairs throughout the system. TouchDesigner sends paired values, the ESP32 parses them as pairs, and the PCA9685 channels map to the physical mechanism layout.

**OSC over UDP.** TouchDesigner sends OSC messages to the ESP32's IP address over the local network. For permanent installations, switching to a wired serial connection is more reliable than Wi-Fi.

## Control Modes

- **Gesture:** Leap Motion tracks hand position in real time. TouchDesigner maps it to servo targets and streams values over UDP.
- **Manual:** Slider UI gives direct control over the X and Y axes for all servo pairs.
- **Animated:** TouchDesigner's Animation COMP drives repeatable keyframe sequences. A Python script inside the `.toe` file generates the keyframe table from a pose table, so sequences are defined as data rather than manually edited curves.

## Repository Contents

**ESP32/**
- `touchdesigner-udp-com.ino`: Connects to Wi-Fi, listens for OSC messages, and controls the X/Y servo pairs via the PCA9685.

**TouchDesigner/**
- `motor-control-system_005v2-1.toe`: Control network with switchable inputs for Leap Motion, manual sliders, Animation COMP, and a reset position.

## Hardware

**ESP32** development board

**PCA9685** 16-channel PWM servo driver

**EMAX ES08MA II** hobby servos, or similar

**Leap Motion Controller** gesture sensor

**5 V power supply** for servo power rail

## Software

- **TouchDesigner**
- **Arduino IDE** with the ESP32 board manager installed
- **Arduino libraries:** `Adafruit_PWMServoDriver`, `OSC`

## Setup

1. Open `touchdesigner-udp-com.ino` and set your Wi-Fi `ssid` and `password`.
2. Upload the sketch to the ESP32.
3. Open `motor-control-system_005v2-1.toe` in TouchDesigner. Update the IP address in the `OSC Out CHOP` to match the ESP32.
4. Use the mode buttons in the network to switch between Leap Motion, manual, and animated control through the `switch_inputMaster` CHOP.

## Keyframe Generation

The `.toe` file includes a Python script called `generate_keys_script`. It reads servo poses from the `null_table_keys` table and generates the correctly formatted `keys` table for the Animation COMP. Defining poses as table rows is faster than editing animation curves by hand and makes sequences easier to rearrange or extend.

## Links

- [Project Write-up](https://stevenmbenton.com/hobby-servo-control-system/)
- [Gesture-Controlled 2-Axis Mechanisms](https://youtu.be/UZ0vq4jCJZ0)
