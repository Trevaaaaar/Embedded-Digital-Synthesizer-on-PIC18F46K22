# Embedded Digital Synthesizer on PIC18F46K22

## Overview

This project is a bare-metal embedded digital synthesizer built on a PIC18F46K22 microcontroller. The synthesizer generates real-time audio using direct digital synthesis and outputs samples through an MCP4822 12-bit SPI DAC. User-adjustable sound parameters are controlled through potentiometers, push buttons, and a 16x2 LCD menu interface.

The system supports multiple waveforms, chromatic pitch selection, volume control, ADSR-style envelope shaping, tremolo modulation, and EEPROM-backed preset storage. The firmware is organized around a fast timer interrupt for audio generation and a slower control loop for ADC input, LCD updates, button debouncing, and preset management.

## Features

- Real-time audio output at a 10 kHz sample rate
- DDS waveform generation using a 32-bit phase accumulator
- 256-sample sine wavetable with additional square, sawtooth, and triangle waveforms
- MCP4822 12-bit DAC controlled over SPI
- ADC-based control of volume, waveform, envelope, tremolo, and pitch
- LCD menu interface for editing sound parameters
- ADSR-style envelope with attack, decay, sustain, and release parameters
- Tremolo modulation using a low-frequency oscillator
- Structured note profile system for sound parameters
- EEPROM preset save/load for persistent settings

## System Architecture

The synthesizer separates real-time audio generation from slower user-interface tasks.

The Timer2 interrupt runs at the audio sample rate. Each interrupt updates the DDS phase accumulator, selects the current waveform sample, applies envelope and tremolo scaling, and sends the final sample to the MCP4822 DAC over SPI.

The main loop handles slower tasks such as reading potentiometers, updating the LCD menu, debouncing buttons, and applying changes to the current note profile. This separation prevents slow UI operations from interfering with the real-time audio output.

## Hardware

| Component | Purpose |
| --------- | ------- |
| PIC18F46K22 | Main microcontroller |
| MCP4822 | 12-bit SPI DAC for audio output |
| 16x2 LCD | Menu and parameter display |
| Potentiometers | Analog parameter control |
| Push buttons | Note trigger and menu navigation |
| Audio jack / amplifier | Audio output |

## Timing and Performance
| Parameter | Value |
|---|---|
| MCU clock | 64 MHz |
| Audio sample rate | 10 kHz |
| Audio timer | Timer2, 100 us tick |
| Control/debounce timer | Timer0, 1 ms tick |
| DAC resolution | 12-bit |
| DDS accumulator | 32-bit |
| Wavetable size | 256 samples |
| Pitch range | C4 to C5 chromatic octave |

## User Interface

The LCD menu is divided into parameter screens. Two potentiometers adjust the active parameters depending on the selected screen.

| Screen | Pot 1 | Pot 2 |
|---|---|---|
| 0 | Volume | Waveform |
| 1 | Attack | Decay |
| 2 | Sustain | Release |
| 3 | Tremolo depth | Pitch |
| 4 | Preset No. | Load/Save |

## Challenges and Design Decisions

The main design challenge was generating audio at a fixed sample rate while also handling user input and LCD updates simultaneously. To prevent slow UI operations from affecting audio timing, the firmware separates the fast audio path from the slower control path. Timer2 drives the 10 kHz audio interrupt, where the DDS phase accumulator is updated, a waveform sample is generated, envelope and tremolo scaling are applied, and the final sample is sent to the MCP4822 DAC over SPI.

The synth DDS uses the upper 8 bits of a 32-bit phase accumulator as a dynamic index that moves through a 256-sample wavetable. This allows for pitch to be controlled by changing the phase increment instead of the timer period and for easy calculation of samples for square, sawtooth, and triangle waves.

ADC readings from the potentiometers did not always span the full 0–1023 range on the breadboard, so the firmware calibrates the measured range and maps it into user-facing values such as percentages for volume/tremolo and milliseconds for ADSR timing. The LCD menu screens allow for two potentiometers to control different pairs of sound settings and allow for modular expansion with more parameters. The menu edit functionality is necessary so that sound parameter values are not automatically updated upon switching menu screens. 

User-interface tasks are handled separately in the main loop using a slower Timer0-based control tick. This includes ADC reads, LCD menu updates, button debouncing, and parameter editing. Sound parameters are stored in a structured note profile, which made it easier to organize volume, waveform, pitch, envelope, and tremolo settings. UI parameters and audio parameters were separated so the LCD/ADC control logic could update settings without the audio ISR reading partially modified values.

Several design decisions were made to fit the constraints of the PIC18F46K22. The audio path uses integer arithmetic instead of floating-point math, the LCD is updated only when needed, and button inputs are debounced using counters rather than blocking delays. These choices helped keep the audio interrupt predictable while still allowing real-time control of the synth.

## Testing and Verification
The synthesizer was tested and documented incrementally by verifying various hardware and firmware subsystems before integrating everything together, reducing debugging headache and providing isolated code samples that are more easily understood. Testing began with basic DAC communication, then progressed through audio waveform generation, timer-driven sampling, DDS pitch control, envelope shaping, tremolo, LCD/ADC input, button debouncing, and final system integration.

| Stage | Test / Milestone | What Was Verified |
|---|---|---|
| 1 | MCP4822 DAC SPI test | Confirmed that the PIC18F46K22 could send valid SPI commands and set a stable DAC output voltage. |
| 2 | 440 Hz square wave output | Verified the basic audio path by toggling the DAC between low and high output values. |
| 3 | 64-sample sine wave test | Confirmed wavetable-based audio output using a sine lookup table and software delay timing. |
| 4 | Timer2 interrupt-driven sine wave | Replaced software delays with hardware-timed sample output from a Timer2 ISR. |
| 5 | DDS audio generation | Verified 32-bit phase-accumulator synthesis with a fixed 10 kHz sample rate and 256-sample wavetable. |
| 6 | Pitch and waveform selection | Confirmed chromatic note stepping and sine, square, sawtooth, and triangle waveform generation from the same DDS engine. |
| 7 | Volume control and D0 note trigger | Verified digital amplitude scaling and button-controlled note triggering. |
| 8 | Envelope implementation | Tested attack/decay behavior first, then expanded to full ADSR with note-on and note-off release behavior. |
| 9 | Tremolo / LFO modulation | Verified low-frequency amplitude modulation using a hardcoded LFO sine table and optimized ISR gain calculation. |
| 10 | Timer0 debounce and control tick | Added a 1 ms Timer0 tick to handle button debouncing and slower control tasks separately from the audio ISR. |
| 11 | ADC and LCD menu tests | Verified potentiometer input, LCD display output, menu navigation, edit mode, and parameter screen updates. |
| 12 | LCD/menu optimization | Reduced LCD flicker and unnecessary display writes using a dirty flag and slower refresh interval. |
| 13 | Full audio/UI merge | Confirmed that audio generation continued while LCD menu controls adjusted volume, waveform, envelope, tremolo, and pitch. |
| 14 | Note profile struct | Grouped volume, waveform, attack, decay, sustain, release, tremolo depth, and pitch into a reusable sound-profile structure. |
| 15 | UI/audio parameter separation | Separated UI-edited parameters from committed audio-engine parameters to prevent unsafe ISR reads during updates. |
| 16 | EEPROM preset save/load | Verified that note profile settings could be saved to EEPROM, reloaded, and retained after reset or power cycling. |
| 17 | ADC calibration | Remapped measured potentiometer ADC limits to the full software range so endpoints like 0%, 100%, C4, and C5 were reachable. |
| 18 | Final integrated system test | Verified real-time audio output with LCD control, calibrated ADC input, ADSR, tremolo, pitch selection, and EEPROM-backed presets working together. |

## Future Improvements
- Add polyphonic note support
- Implement 12-button matrix as a piano-like input interface
- Improve enclosure and control layout
- Add MIDI input
- Add additional modulation effects such as vibrato
