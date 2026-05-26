# Embedded-Digital-Synthesizer-on-PIC18F46K22

## Overview

This project is a bare-metal embedded digital synthesizer built on a PIC18F46K22 microcontroller. The synthesizer generates real-time audio using direct digital synthesis and outputs samples through an MCP4822 12-bit SPI DAC. User-adjustable sound parameters are controlled through potentiometers, push buttons, and a 16x2 LCD menu interface.

The system supports multiple waveforms, chromatic pitch selection, volume control, ADSR-style envelope shaping, tremolo modulation, and EEPROM-backed preset storage. The firmware is organized around a fast timer interrupt for audio generation and a slower control loop for ADC input, LCD updates, button debouncing, and preset management.
