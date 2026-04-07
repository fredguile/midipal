# MIDIpal — Fred Guile Custom Firmware 1.5

Fork of the [MIDIpal](https://github.com/pichenettes/midipal) MIDI Swiss army knife by Emilie Gillet, with custom enhancements.

## What is MIDIpal?

A MIDI in, a MIDI out, a small display, an encoder; 20 awesome features:

- MIDI monitor
- BPM counter
- Keyboard splitter
- Note dispatcher / voice stealing / polyphonic driver
- Channels merger
- **Channel filter** (enhanced — see below)
- Clock divider
- Clock source, with swing
- CC and NRPN knob
- 8 pots/sensors voltage reader/controller
- Drum pattern generator (including Euclidian patterns)
- Note, velocity and CC randomizer
- Chord memory
- Arpeggiator
- Sequencer
- CC LFO
- Tempo-synchronized note delay (with transposition and decay velocity)
- Slave start/stop synchronizer (a la Mungo Sync)
- Scale processor / harmonizer
- Ear training game

## Changes from Upstream

### Channel Filter Enhancements

The **filter** app (formerly "chnfiltr") has been improved:

- **Renamed**: Display label changed from `chnfiltr` to `filter`
- **MIDI Start/Stop/Continue filtering**: New toggle (`strstp`) lets you block or pass MIDI Start (0xFA), Continue (0xFB), and Stop (0xFC) messages. This complements the existing clock filter toggle (`clk`).

The filter settings page order is now:

| Page | Label          | Description                                    |
| ---- | -------------- | ---------------------------------------------- |
| 1    | `clk`          | MIDI Clock pass-through (ON/OFF)               |
| 2    | `strstp`       | MIDI Start/Stop/Continue pass-through (ON/OFF) |
| 3–18 | `ch 1`–`ch 16` | Per-channel pass-through (ON/OFF)              |

All toggles follow the same convention: **ON = messages pass through**, **OFF = messages are filtered**.

### avr-gcc 9+ Compatibility

The original firmware targeted avr-gcc 4.x (CrossPack era) and would silently miscompile with modern toolchains. This fork fixes compatibility with avr-gcc 9.x and later, as shipped by current Linux distributions and Homebrew (`osx-cross/avr`).

Key fixes include:

- Replaced deprecated `prog_*` PROGMEM typedef aliases (`prog_char`, `prog_uint8_t`, etc.) with standard `PROGMEM` variable declarations — GCC 9 silently ignores `PROGMEM` on typedefs, causing flash data to be read as garbage from RAM
- Added explicit `return` statements to all non-void functions — GCC 9 treats missing returns as undefined behavior and may optimize away entire function bodies
- Moved LCD driving from timer ISR to the main loop to avoid reentrancy issues with GCC 9 optimizations

## Building

### Prerequisites

Install the AVR toolchain for your system:

**Ubuntu/Debian:**

```bash
sudo apt install gcc-avr avr-libc avrdude
```

**Fedora:**

```bash
sudo dnf install avr-gcc avr-libc avrdude
```

**Arch Linux:**

```bash
sudo pacman -S avr-gcc avr-libc avrdude
```

**macOS (Apple Silicon):**

```bash
brew tap osx-cross/avr
brew install avr-gcc avrdude
```

Python is also required (usually already installed).

### Build

Build the firmware:

```bash
make AVRLIB_TOOLS_PATH=/usr/bin/
```

The compiled firmware will be output to `build/midipal/midipal.hex`.

**macOS (Apple Silicon):**

```bash
make AVRLIB_TOOLS_PATH=/opt/homebrew/bin/
```

**macOS (Intel):**

```bash
make AVRLIB_TOOLS_PATH=/usr/local/bin/
```

### Other Make Targets

| Command                                   | Description                             |
| ----------------------------------------- | --------------------------------------- |
| `make AVRLIB_TOOLS_PATH=/usr/bin/ midi`   | Generate MIDI file for firmware update  |
| `make AVRLIB_TOOLS_PATH=/usr/bin/ syx`    | Generate SysEx file for firmware update |
| `make AVRLIB_TOOLS_PATH=/usr/bin/ size`   | Show code size statistics               |
| `make AVRLIB_TOOLS_PATH=/usr/bin/ upload` | Flash firmware via ISP programmer       |

### Flashing

There are two ways to flash firmware to MIDIpal:

#### Method 1: Using a Programmer (ISP)

Connect your programmer and run:

```bash
make AVRLIB_TOOLS_PATH=/usr/bin/ upload
```

Adjust `PROGRAMMER` and `PROGRAMMER_PORT` variables in `avrlib/makefile.mk` if needed.

#### Method 2: Using SysEx or MIDI File

You can update the firmware without a programmer by sending it via MIDI.

1. **Generate the firmware file:**
   - For SysEx: `make AVRLIB_TOOLS_PATH=/usr/bin/ syx` produces `build/midipal/midipal.syx`
   - For MIDI file: `make AVRLIB_TOOLS_PATH=/usr/bin/ midi` produces `build/midipal/midipal.mid`

2. **Put MIDIpal in update mode:**
   - Keep the encoder pressed while powering on MIDIpal
   - You'll see MIDI in and MIDI out LEDs rapidly blink in sequence
   - The MIDI in LED will stay on — the device is ready to receive the update

3. **Send the firmware:**
   - Send the `.syx` or `.mid` file to MIDIpal's MIDI IN
   - Use a 250 ms (or more) delay between SysEx packets
   - The MIDI in LED blinks on every received packet
   - Update takes about 1 minute (up to 256 packets)
   - The update can be restarted in case of an accident during transmission

4. **Completion:**
   - Upon reception of the last packet, MIDIpal immediately boots with the new firmware

> **Tip:** For SysEx transfers, we recommend [Elektron's C6 tool](https://www.elektron.se/support-and-downloads) with proper timing settings.

## Important Notes

- This custom firmware expands the filter app settings from 17 to 18 bytes. If upgrading from the upstream firmware, perform a **factory reset** after flashing (select `!reset!` in the app selector) to ensure settings are properly initialized.

## License

This project is licensed under the GNU General Public License v3.0 — see the source files for details.
