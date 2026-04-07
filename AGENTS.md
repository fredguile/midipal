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

## GCC 9+ Compatibility Notes (macOS / osx-cross/avr)

The codebase originally targeted avr-gcc 4.x (CrossPack era). When building
with avr-gcc 9.x (e.g. via `brew install avr-gcc` using the `osx-cross/avr`
tap), the following rules must be followed to avoid silent miscompilation:

### 1. Never use deprecated `prog_*` PROGMEM typedef aliases
`prog_char`, `prog_uint8_t`, `prog_uint16_t` etc. are defined in
`<avr/pgmspace.h>` only when `__PROG_TYPES_COMPAT__` is defined. In GCC 9,
placing `PROGMEM` on a **typedef** is silently ignored. This means pointer
types like `const prog_uint16_t*` are treated as plain RAM pointers, causing
`pgm_read_word()` / `pgm_read_byte()` to read garbage from RAM instead of flash.

**Rule:** Always place `PROGMEM` on the **variable declaration**, not on the type:
```cpp
// ✅ Correct
const uint16_t my_table[] PROGMEM = { ... };
const uint16_t* PROGMEM table_of_tables[] = { ... };

// ❌ Wrong (GCC 9 silently ignores PROGMEM on typedef)
const prog_uint16_t my_table[] = { ... };
const prog_uint16_t* table_of_tables[] = { ... };
```

### 2. Always add return statements to non-void functions
GCC 9 treats missing `return` in a non-void function as **undefined behavior**
and may optimize away the entire function body. This was responsible for
`Lcd::WriteData()` and `Lcd::WriteCommand()` silently discarding all LCD writes.

**Rule:** Every non-void code path must have an explicit `return` statement.

### 3. Avoid ISR_NOBLOCK
Re-entrant ISRs (`ISR_NOBLOCK`) cause unpredictable behavior with GCC 9
optimization. Avoid them — use the main loop for non-time-critical work
instead.

### 4. LCD driving runs in the main loop
`display.Tick()` + `Lcd::Flush()` are called from the main loop (not from
the timer ISR) to ensure safe, non-reentrant LCD updates. `Lcd::Flush()`
uses `SlowWrite` to drain the output buffer. Call `display.Tick()` 8 times
per main loop iteration (once per display position) to ensure all characters
are refreshed.

### 5. macOS Build Commands
```bash
# Install toolchain (macOS)
brew tap osx-cross/avr
brew install avr-gcc avrdude

# Apple Silicon (M1/M2/M3)
make AVRLIB_TOOLS_PATH=/opt/homebrew/bin/

# Intel Mac
make AVRLIB_TOOLS_PATH=/usr/local/bin/

# Build + flash everything (fuses + bootloader + app + EEPROM)
make AVRLIB_TOOLS_PATH=/opt/homebrew/bin/ bake_all
```
