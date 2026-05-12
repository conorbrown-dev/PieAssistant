# Raspberry Pi Voice Assistant Tutorial Plan

## Summary
Build the project as a **Python-based, on-device voice assistant** for a **Raspberry Pi 4/5** using a **USB microphone** for input and **powered speakers or an amplified speaker path** on the 3.5 mm jack for output. The tutorial should teach through milestones, not hand over a finished app: every example stays inside comments or pseudocode-style comment blocks, with the reader implementing the real code.

The end-state interaction is:
1. Conor says the wake phrase.
2. The assistant speaks the function groups: `Weather Reports`, `Clock`, `ToDo Items`, `Reminders`, `Timers`.
3. Conor says a group such as `Weather Reports`.
4. The assistant speaks the functions in that group.
5. Conor says `Execute function Full Report`.
6. The assistant reads the weather sections with deliberate pauses between `High and Low Temps`, `Humidity`, and `Precipitation`.

## Key Design Choices
- **Language**: Python.
Reason: best fit for Raspberry Pi GPIO/Linux workflows, mature audio tooling, and strong local speech ecosystem.
- **Audio I/O**: use `sounddevice` over PortAudio as the Python-facing audio layer, with ALSA/PipeWire underneath on Raspberry Pi OS.
- **Wake phrase**: use `openWakeWord` for the hands-free milestone.
- **Speech-to-text**: use `Vosk` with a constrained vocabulary for menu navigation and command execution.
- **Text-to-speech**: use `Piper` for local spoken replies.
- **Weather provider**: use `Open-Meteo` for the tutorial path, but keep the weather layer isolated so a later swap to another provider is easy.
- **Project rule**: no runnable code in the tutorial. Only commented interface examples, sequence examples, and data-shape examples.

## Public Interfaces And Tutorial-Defined Types
The tutorial should define these conceptual interfaces early, with comment-only examples:
- `AudioInput`
A component that yields short PCM frames from the USB microphone.
- `WakeWordDetector`
Consumes audio frames and emits `wake_detected`.
- `SpeechRecognizer`
Consumes post-wake audio and returns one recognized phrase.
- `DialogueState`
States: `idle`, `announce_groups`, `await_group`, `announce_functions`, `await_function`, `execute_function`, `speak_result`.
- `FunctionCatalog`
A data shape that maps group names to callable functions and spoken descriptions.
- `ResponseSpeaker`
Takes text segments plus pause timing instructions and plays spoken audio.
- `WeatherService`
Returns a normalized weather result such as high/low, humidity, precipitation, and full report text.

The tutorial should include comment-only examples of:
- microphone frame capture
- feeding frames into wake-word detection
- running speech recognition on a captured utterance
- routing recognized text to a function group or function
- synthesizing spoken replies with pauses between sections

## Implementation Plan
- **Milestone 1: Hardware and OS grounding**
Use Raspberry Pi OS, verify the USB microphone is selected as input, and route output to the 3.5 mm jack. Teach how Linux audio devices are exposed and why powered speakers matter.
- **Milestone 2: Audio pipeline mental model**
Explain the real-time loop: listen continuously, detect wake phrase, capture a command utterance, recognize speech, decide intent, speak a reply, return to idle.
- **Milestone 3: Dialogue tree before features**
Have the tutorial define the exact spoken menu flow and dialogue states before adding any real feature logic. This is where the function-group hierarchy is designed.
- **Milestone 4: Local speech stack**
Introduce `openWakeWord`, `Vosk`, and `Piper` one at a time. Explain what each stage is responsible for and how latency, false activations, and vocabulary constraints affect behavior.
- **Milestone 5: Function catalog**
Start with `Weather Reports`, then stub the other groups as empty-but-announced categories. `Weather Reports` should expose `Full Report`, `High and Low Temps`, `Humidity`, and `Precipitation`.
- **Milestone 6: Weather report composition**
Teach how to transform raw weather fields into spoken sections. `Full Report` should assemble all three sections and insert intentional pauses between them for retention.
- **Milestone 7: Reliability and polish**
Cover microphone sensitivity, ambient noise, wake-word threshold tuning, repeat prompts, timeout behavior, and graceful “I didn’t catch that” recovery.

## Test Plan
- Confirm the Pi can record from the USB microphone and play to the intended speaker path without changing devices unexpectedly.
- Confirm the wake phrase only advances the assistant from `idle` to menu mode.
- Confirm each function group is announced clearly and recognized from speech.
- Confirm `Weather Reports` exposes the four expected functions.
- Confirm `Execute function Full Report` produces all three weather sections in the correct order with pauses between them.
- Confirm unrecognized speech returns a retry prompt and does not crash the dialogue loop.
- Confirm the assistant returns to `idle` after completing a reply.
- Confirm the other groups can be announced even if their internal functions are still placeholders.

## Assumptions And Defaults
- Prompt phrase is still TBD; the tutorial should first use an existing `openWakeWord` model, then explain how to swap or train later.
- Assistant name is TBD and should be treated as spoken content/configuration, not a hardcoded architectural dependency.
- Weather is forecast-oriented, not radar/alerts-oriented.
- The tutorial targets a single-user household setup for Conor, not multi-user speaker recognition.
- The tutorial favors clarity and learnability over maximum feature count.
- Audio output on the Pi’s 3.5 mm jack is treated as **line-level**, so the plan assumes **powered speakers or an amplifier**, not passive speakers.

## Sources
- Raspberry Pi audio output behavior: https://www.raspberrypi.com/documentation/computers/getting-started.html
- Raspberry Pi audio routing/configuration: https://www.raspberrypi.com/documentation/computers/configuration.html
- `sounddevice` / PortAudio Python layer: https://pypi.org/project/sounddevice/
- `Vosk` offline speech recognition: https://alphacephei.com/vosk/
- `openWakeWord` wake phrase detection: https://github.com/dscripka/openWakeWord
- `Piper` local TTS: https://github.com/OHF-Voice/piper1-gpl
- `Open-Meteo` forecast API: https://open-meteo.com/en/docs
