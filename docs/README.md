# PieAssistant

This repository is a **DIY tutorial scaffold** for building a Raspberry Pi voice assistant with a USB microphone and speaker output through the Pi's audio path.

It is intentionally **not** a finished assistant. There is no runnable app in this repo. Instead, the project teaches the build in milestones and uses **comment-only examples** so you can implement each piece yourself.

## What You Are Building

The target experience is:

1. Conor says a wake phrase.
2. The assistant replies with the available function groups:
   `Weather Reports`, `Clock`, `ToDo Items`, `Reminders`, `Timers`
3. Conor says a function group, such as `Weather Reports`.
4. The assistant replies with the functions inside that group.
5. Conor says `Execute function Full Report`.
6. The assistant reads the weather report in sections, pausing between ideas so the information is easier to retain.

## Project Principles

- This is a learning project first.
- The repo should help you build it, not build it for you.
- No actual runnable assistant code is included.
- All code examples appear only as comments inside example blocks.
- The recommended stack is Python on Raspberry Pi OS with local speech components.

## Recommended Stack

- **Language**: Python
- **Audio I/O**: `sounddevice` over PortAudio
- **Wake phrase**: `openWakeWord`
- **Speech-to-text**: `Vosk`
- **Text-to-speech**: `Piper`
- **Weather data**: `Open-Meteo`

## How To Use This Repo

Read the docs in order:

1. [Build Tutorial](docs/build-tutorial.md)
2. [Architecture Notes](docs/architecture-notes.md)
3. [Project Structure Guide](docs/project-structure.md)
4. [Commented Examples](docs/commented-examples.md)
5. [Testing Checklist](docs/testing-checklist.md)

Treat each section like a workshop:

- read the concept
- implement it yourself in your own files
- test it on the Pi
- return for the next milestone

## Milestones

1. Hardware and OS grounding
2. Audio pipeline mental model
3. Dialogue tree before features
4. Local speech stack
5. Function catalog
6. Weather report composition
7. Reliability and polish

## Important Hardware Note

The Raspberry Pi 3.5 mm jack is a **line-level output**, not a passive speaker driver. Plan on using:

- powered speakers
- an amplified speaker setup
- or an audio device with its own amplification

## Sources

- Raspberry Pi getting started and audio notes:
  https://www.raspberrypi.com/documentation/computers/getting-started.html
- Raspberry Pi audio routing/configuration:
  https://www.raspberrypi.com/documentation/computers/configuration.html
- `sounddevice`:
  https://pypi.org/project/sounddevice/
- `Vosk`:
  https://alphacephei.com/vosk/
- `openWakeWord`:
  https://github.com/dscripka/openWakeWord
- `Piper`:
  https://github.com/OHF-Voice/piper1-gpl
- `Open-Meteo`:
  https://open-meteo.com/en/docs
