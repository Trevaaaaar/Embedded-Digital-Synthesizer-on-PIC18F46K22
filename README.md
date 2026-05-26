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
- LCD menu interface with edit mode
- ADSR-style envelope with attack, decay, sustain, and release parameters
- Tremolo modulation using a low-frequency oscillator
- Structured note profile system for sound parameters
- EEPROM preset save/load for persistent settings
