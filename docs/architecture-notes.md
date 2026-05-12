# Architecture Notes

This document defines the assistant as a set of responsibilities, not as a finished implementation. Keep these boundaries clear while you build and the project will stay understandable.

## Core Components

### `AudioInput`

Responsibility:

- open the selected USB microphone
- read short PCM frames
- expose a consistent sample format for the rest of the stack

Design note:

Choose one working audio format early and make everything else adapt to it. For local speech tools on Raspberry Pi, `16 kHz`, `mono`, and `16-bit PCM` is the simplest target.

### `WakeWordDetector`

Responsibility:

- consume incoming frames while the assistant is idle
- emit a `wake_detected` signal when the threshold is met

Design note:

This stage should not try to recognize the user's command. Its job is only to decide whether the assistant should start paying close attention.

### `SpeechRecognizer`

Responsibility:

- accept a buffered command utterance after wake
- produce recognized text
- optionally use a constrained vocabulary to improve accuracy

Design note:

For this project, narrow recognition is a feature, not a limitation. The assistant benefits from a small vocabulary because the conversation is menu-driven.

### `FunctionCatalog`

Responsibility:

- describe what groups and functions exist
- provide the spoken names for those items
- connect each leaf function to a behavior you implement later

Design note:

Treat the catalog as the source of truth for what the assistant can advertise. If it is not in the catalog, the assistant should not present it as available.

### `WeatherService`

Responsibility:

- fetch weather data from `Open-Meteo`
- normalize raw fields into a stable internal weather shape
- provide inputs for section-specific and full-report responses

Suggested normalized fields:

- `location_name`
- `forecast_day_label`
- `high_temp`
- `low_temp`
- `humidity_summary`
- `precipitation_summary`

### `ResponseSpeaker`

Responsibility:

- accept reply segments
- synthesize each segment into speech
- enforce pauses between segments

Design note:

Thinking in segments will make your replies easier to test and easier to hear. It also gives you a natural place to insert pauses for retention.

## Dialogue State Machine

The initial state machine should stay small:

- `idle`
- `announce_groups`
- `await_group`
- `announce_functions`
- `await_function`
- `execute_function`
- `speak_result`

### State responsibilities

`idle`

- listen for the wake phrase only

`announce_groups`

- speak the available function groups

`await_group`

- listen for one of the known group names

`announce_functions`

- speak the functions inside the selected group

`await_function`

- listen for a supported function selection command

`execute_function`

- gather data and compose the response

`speak_result`

- play the response and return to `idle`

## Data Flow

1. Audio frames enter through `AudioInput`.
2. Idle frames are evaluated by `WakeWordDetector`.
3. After wake, command audio is buffered.
4. Buffered audio is sent to `SpeechRecognizer`.
5. Recognized text is matched against the `FunctionCatalog`.
6. Selected behavior reads from `WeatherService` or another future service.
7. Output segments are handed to `ResponseSpeaker`.

## Suggested Project Layout

Use the project layout to reinforce the architecture. A beginner-friendly structure for this assistant would look like this:

```text
project-root/
  docs/
  notes/
  config/
  assets/
  models/
  src/
    assistant/
      app.py
      audio/
        input.py
        playback.py
        wake_word.py
        speech_recognition.py
        text_to_speech.py
      dialogue/
        states.py
        router.py
        function_catalog.py
      services/
        weather_service.py
      domain/
        weather_models.py
        response_segments.py
  tests/
```

### Why this layout works

- `audio/` keeps microphone, wake-word, STT, and TTS concerns near each other.
- `dialogue/` owns how the assistant moves between states and routes commands.
- `services/` holds integrations such as weather data providers.
- `domain/` is a good place for normalized data shapes and assistant-facing concepts.
- `config/` gives runtime choices a home outside the code that uses them.
- `notes/` preserves the learning trail, which matters in a DIY build.

### Naming guidance

- package and file names: lowercase snake_case
- classes: PascalCase
- constants: UPPER_SNAKE_CASE
- configuration files: short descriptive names such as `audio.toml` or `assistant.yaml`
- notes: date-prefixed names such as `2026-05-11-audio-routing.md`

### Placement guidance

- Put provider-specific code in `services/`, not in `dialogue/`.
- Put speech model files in `models/` if you want a clearly defined local home for them.
- Put raw scratch experiments in `notes/` or a temporary sandbox folder, then either promote or delete them.
- Avoid a catch-all file such as `utils.py` until a real shared pattern emerges.

## Speech Constraints

Early versions should favor exactness over flexibility.

- exact group names are acceptable
- exact function names are acceptable
- short command phrases are acceptable
- broad free-form conversation is out of scope

This is a good constraint because it keeps the assistant deterministic while you learn the stack.

## Comment-Only Interface Examples

```python
# class AudioInput:
#     # Responsibility:
#     # - open the chosen microphone
#     # - yield 16 kHz mono PCM frames
#     #
#     # Example questions to answer while implementing:
#     # - how large should each frame be?
#     # - how will device selection be configured?
#     # - what happens if the device disappears?
#
# class WakeWordDetector:
#     # Responsibility:
#     # - read frames from AudioInput
#     # - decide whether the wake phrase was spoken
#     #
#     # Example implementation direction:
#     # - keep this active only in the idle state
#     # - tune threshold against your real room noise
#
# class SpeechRecognizer:
#     # Responsibility:
#     # - turn one captured utterance into text
#     #
#     # Example implementation direction:
#     # - begin with a constrained grammar for:
#     #   Weather Reports
#     #   Clock
#     #   ToDo Items
#     #   Reminders
#     #   Timers
#     #   Execute function Full Report
#
# class ResponseSpeaker:
#     # Responsibility:
#     # - speak one or more text segments
#     # - pause between sections when requested
```

## Weather Group Behavior

### `High and Low Temps`

Reads the normalized daily high and low for the chosen forecast day.

### `Humidity`

Reads a concise humidity summary. Prefer one spoken number or a short range over a long technical explanation.

### `Precipitation`

Reads a concise precipitation summary. Keep the wording consistent so it is easy to compare day to day.

### `Full Report`

Composes the three sections in a fixed order:

1. High and low temperatures
2. Humidity
3. Precipitation

Each section should be spoken as its own unit with a pause between units.
