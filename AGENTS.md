# Agent Instructions for Midipal Project

This file contains instructions for AI agents working on the Midipal project.

## Build Instructions

**IMPORTANT:** All build commands must use the `AVRLIB_TOOLS_PATH` parameter:

### Build Firmware
```bash
make AVRLIB_TOOLS_PATH=/usr/bin/
```

Output: `build/midipal/midipal.hex`

### Generate MIDI File for Firmware Update
```bash
make AVRLIB_TOOLS_PATH=/usr/bin/ midi
```

### Generate SysEx File for Firmware Update
```bash
make AVRLIB_TOOLS_PATH=/usr/bin/ syx
```

### Show Code Size Statistics
```bash
make AVRLIB_TOOLS_PATH=/usr/bin/ size
```

### Flash Firmware to Device
```bash
make AVRLIB_TOOLS_PATH=/usr/bin/ upload
```

## Prerequisites

The project requires AVR toolchain installed:
- **Ubuntu/Debian:** `sudo apt install gcc-avr avr-libc avrdude`
- **Fedora:** `sudo dnf install avr-gcc avr-libc avrdude`
- **Arch Linux:** `sudo pacman -S avr-gcc avr-libc avrdude`

Python is also required.

## Project Structure

- `midipal/` - Main firmware source code
- `midipal/resources/` - Resources (strings, lookup tables) that can be regenerated
- `avrlib/` - AVR library
- `build/midipal/` - Build output directory

## Version String

The version string is defined in:
1. `midipal/makefile` (VERSION variable)
2. `midipal/resources/strings.py` (line 49)
3. `midipal/resources.cc` (generated file, can be edited manually)

When changing the version string, remember it must fit in 8 characters on the LCD display.
