# Embedded-Digital-Synthesizer-on-PIC18F46K22

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

ADC reading with potentiometers on noisy breadboards sometimes did not deliver the full range from 0 to 1023, so the ADC values were calibrated and converted into different forms like percentages for volume and tremolo or millisecond values for the attack, decay, and release parameters. The LCD menu screens allow for two potentiometers to control different pairs of sound settings and allow for modular expansion with more parameters. The menu edit functionality is necessary so that sound parameter values are not automatically updated upon switching menu screens. 

User-interface tasks are handled separately in the main loop using a slower Timer0-based control tick. This includes ADC reads, LCD menu updates, button debouncing, and parameter editing. Sound parameters are stored in a structured note profile, which made it easier to organize volume, waveform, pitch, envelope, and tremolo settings. UI and audio parameters were separated into different note profiles to make code easier to manage and behave more predictably. 

Several design decisions were made to fit the constraints of the PIC18F46K22. The audio path uses integer arithmetic instead of floating-point math, the LCD is updated only when needed, and button inputs are debounced using counters rather than blocking delays that burden the CPU. These choices helped keep the audio interrupt predictable while still allowing real-time control of the synth.

## Testing and Verification
The synthesizer was tested and documented incrementally by verifying various hardware and firmware subsystems before integrating everything together, reducing debugging headache and providing isolated code samples that are more easily understood. Testing began with basic DAC communication, then progressed through audio waveform generation, timer-driven sampling, DDS pitch control, envelope shaping, tremolo, LCD/ADC input, button debouncing, and final system integration.

| Test Stage | Code Test / Milestone                            | Purpose                                                  | Verification Method                                                                    | Result                                                                                   |
| ---------- | ------------------------------------------------ | -------------------------------------------------------- | -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| 1          | MCP4822 12-bit DAC test code                     | Verify SPI communication with the external DAC           | Sent a fixed DAC command to MCP4822 channel A                                          | Confirmed that the DAC output could be set to a stable analog voltage                    |
| 2          | MCP4822 square wave audio test, 440 Hz           | Verify basic audio output path                           | Alternated DAC output between low and high values using software delays                | Produced an audible square-wave tone                                                     |
| 3          | MCP4822 sine wave audio test, 440 Hz, 64 samples | Verify wavetable-based audio generation                  | Generated a 64-sample sine lookup table and stepped through it with fixed delays       | Confirmed that the DAC could output a repeating sine-like waveform                       |
| 4          | Sine wave audio + Timer2                         | Replace software delay timing with hardware timer timing | Used Timer2 interrupt to output sine samples at a fixed interval                       | Confirmed interrupt-driven audio generation                                              |
| 5          | Sine wave audio + DDS                            | Verify direct digital synthesis                          | Used a 32-bit phase accumulator and wavetable lookup to generate pitch                 | Confirmed pitch generation without changing the sample timer                             |
| 6          | DDS + button input to increase pitch             | Verify interactive pitch control                         | Used a button to cycle through phase increment values                                  | Confirmed chromatic pitch stepping from stored note increments                           |
| 7          | Pitch button + wave type button + triangle wave  | Verify waveform selection                                | Added sine, square, sawtooth, and triangle output modes                                | Confirmed multiple waveforms could be selected from the same DDS phase accumulator       |
| 8          | Volume control + D0 piano key                    | Verify amplitude scaling and note triggering             | Added volume levels and used D0 as a note trigger                                      | Confirmed that output amplitude could be scaled and notes could be gated by button input |
| 9          | Attack-decay-idle envelope + D0 piano key        | Verify basic envelope state machine                      | Added attack and decay states triggered by note-on events                              | Confirmed that note amplitude could change over time instead of turning on instantly     |
| 10         | ADSR envelope + D0 piano key                     | Verify full envelope behavior                            | Added attack, decay, sustain, and release states                                       | Confirmed note-on and note-off behavior with release fadeout                             |
| 11         | Constant tremolo + ISR optimization + LFO table  | Verify low-frequency amplitude modulation                | Added an LFO sine table and tremolo gain calculation                                   | Confirmed amplitude modulation while keeping audio ISR math efficient                    |
| 12         | Timer0 debounce + envelope out of ISR            | Verify separation of fast and slow tasks                 | Used Timer0 as a 1 ms control tick for debounce and envelope updates                   | Reduced blocking delays and moved slower control tasks away from the audio ISR           |
| 13         | 2x2 button matrix tests                          | Evaluate expanded note-input hardware                    | Scanned a 2x2 button matrix and mapped keys to note indices                            | Verified matrix input concept, but final design moved away from this interface           |
| 14         | Potentiometer ADC + LCD display test             | Verify ADC and LCD independently                         | Read potentiometer values and displayed raw ADC values on LCD                          | Confirmed analog input and LCD display operation                                         |
| 15         | Pot ADC + LCD + menu test                        | Verify multi-screen parameter editing                    | Used potentiometers to update different parameters depending on menu screen            | Confirmed basic menu navigation and parameter display                                    |
| 16         | LCD menu finished / optimized test               | Verify stable UI update behavior                         | Added edit mode, LCD dirty flag, slower LCD refresh, and debounced screen/edit buttons | Reduced flicker and confirmed reliable menu interaction                                  |
| 17         | Synth code merged with LCD/pot code              | Verify combined audio and UI operation                   | Integrated audio engine, LCD menu, ADC controls, and button input                      | Confirmed that sound generation and UI control could run together                        |
| 18         | Pitch control on tremolo screen                  | Verify ADC-controlled note selection                     | Mapped potentiometer input to 13 note indices from C4 to C5                            | Confirmed pitch could be selected from the LCD menu                                      |
| 19         | Note profile struct implementation               | Verify structured sound parameter storage                | Stored volume, waveform, envelope, tremolo, and pitch in a `note_profile` struct       | Improved organization and prepared the code for presets                                  |
| 20         | UI/audio parameter separation                    | Verify safer real-time parameter updates                 | Separated UI-edited parameters from audio-engine parameters                            | Prevented the audio ISR from reading partially updated UI values                         |
| 21         | LCD startup reset                                | Verify predictable startup state                         | Forced LCD menu to start at screen 0 with edit mode disabled                           | Confirmed consistent behavior after programming/reset                                    |
| 22         | Calibrated ADC                                   | Verify full parameter range access                       | Added ADC min/max calibration and remapping functions                                  | Made endpoints such as 0%, 100%, C4, and C5 easier to reach                              |


## Future Improvements
- Add polyphonic note support
- Implement 12-button matrix as a piano-like input interface
- Improve enclosure and control layout
- Add MIDI input
- Add additional modulation effects such as vibrato
