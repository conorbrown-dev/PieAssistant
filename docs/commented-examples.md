# Commented Examples

Everything in this file is intentionally written as comments so you can study the interface shape without copying a runnable implementation.

## Suggested File Map

```python
# Suggested future implementation map:
#
# src/assistant/app.py
#     - top-level runtime entrypoint you create later
#
# src/assistant/audio/input.py
#     - microphone device selection
#     - frame capture
#
# src/assistant/audio/wake_word.py
#     - openWakeWord integration
#     - wake threshold tuning hooks
#
# src/assistant/audio/speech_recognition.py
#     - Vosk recognizer setup
#     - utterance transcription flow
#
# src/assistant/audio/text_to_speech.py
#     - Piper synthesis integration
#     - playback handoff
#
# src/assistant/audio/playback.py
#     - speaker output selection
#     - segment playback and pause timing
#
# src/assistant/dialogue/states.py
#     - dialogue state names
#     - valid transitions
#
# src/assistant/dialogue/router.py
#     - map recognized text to groups and functions
#
# src/assistant/dialogue/function_catalog.py
#     - source of truth for groups and functions
#
# src/assistant/services/weather_service.py
#     - Open-Meteo fetch and normalization
#
# src/assistant/domain/weather_models.py
#     - normalized weather shapes you define
#
# src/assistant/domain/response_segments.py
#     - spoken segment shape such as text plus pause duration
```

## Microphone Capture

```python
# import sounddevice as sd
#
# Goal:
# - open the USB microphone as an input stream
# - request 16 kHz mono 16-bit style audio
# - read small frames that are suitable for wake-word detection
#
# Possible implementation shape:
# - choose the input device by explicit device id or name match
# - open an input stream with a small block size
# - convert frames into the exact format expected by the wake-word and STT layers
#
# Questions to answer while implementing:
# - will you pull frames in a loop or receive them in a callback?
# - how will you buffer audio after wake detection?
# - what will you log when device open fails?
```

## Wake Phrase Detection

```python
# from openwakeword.model import Model
#
# Goal:
# - keep the detector active only while the assistant is idle
# - feed it 16 kHz PCM audio frames
# - switch state when the wake phrase score crosses a chosen threshold
#
# Possible implementation shape:
# - instantiate one wake model
# - for each audio frame:
#     - ask the model for a score
#     - compare the score against your threshold
#     - if positive, emit wake_detected and begin command capture
#
# Tuning ideas:
# - increase threshold if the room causes false activations
# - lower threshold if the assistant misses your real voice too often
# - test from the real distance where you plan to use it
```

## Command Capture And Speech Recognition

```python
# from vosk import Model, KaldiRecognizer
#
# Goal:
# - capture one utterance after wake
# - stop capture on silence or timeout
# - recognize from a constrained vocabulary first
#
# Example vocabulary to start with:
# - Weather Reports
# - Clock
# - ToDo Items
# - Reminders
# - Timers
# - Full Report
# - High and Low Temps
# - Humidity
# - Precipitation
# - Execute function Full Report
#
# Possible implementation shape:
# - initialize Vosk with a local English model
# - create a recognizer configured for the assistant sample rate
# - after wake, stream the buffered command audio into the recognizer
# - read the recognized text result
# - normalize capitalization and spacing before routing
```

## Dialogue Routing

```python
# Goal:
# - route recognized text according to the current dialogue state
#
# Example state-aware routing idea:
# - if state is await_group:
#     - match recognized text against known group names
#     - if matched, move to announce_functions
#     - if not matched, speak a retry prompt
# - if state is await_function:
#     - look for an execute phrase plus a known function name
#     - if matched, move to execute_function
#     - if not matched, speak a retry prompt
#
# Design advice:
# - normalize text before matching
# - keep accepted phrases narrow at first
# - log every mismatch so you can see what you actually said versus what was recognized
```

## Function Catalog Shape

```python
# FUNCTION_CATALOG = {
#     "Weather Reports": {
#         "spoken_label": "Weather Reports",
#         "functions": {
#             "Full Report": {
#                 "spoken_label": "Full Report",
#                 "behavior": "compose the three weather sections in order",
#             },
#             "High and Low Temps": {
#                 "spoken_label": "High and Low Temps",
#                 "behavior": "read normalized high and low temperatures",
#             },
#             "Humidity": {
#                 "spoken_label": "Humidity",
#                 "behavior": "read normalized humidity summary",
#             },
#             "Precipitation": {
#                 "spoken_label": "Precipitation",
#                 "behavior": "read normalized precipitation summary",
#             },
#         },
#     },
#     "Clock": {
#         "spoken_label": "Clock",
#         "functions": {
#             # Add these yourself later.
#         },
#     },
#     "ToDo Items": {
#         "spoken_label": "ToDo Items",
#         "functions": {
#             # Add these yourself later.
#         },
#     },
#     "Reminders": {
#         "spoken_label": "Reminders",
#         "functions": {
#             # Add these yourself later.
#         },
#     },
#     "Timers": {
#         "spoken_label": "Timers",
#         "functions": {
#             # Add these yourself later.
#         },
#     },
# }
```

## Weather Provider Boundary

```python
# Goal:
# - isolate provider-specific data from spoken-response logic
#
# Example normalization idea:
# - raw API response comes in from Open-Meteo
# - extract only the fields needed for the assistant
# - convert them into one normalized weather object
#
# Example normalized weather shape:
# - location_name
# - forecast_day_label
# - high_temp
# - low_temp
# - humidity_summary
# - precipitation_summary
#
# Reason for this boundary:
# - if you later switch to weather.gov or another provider,
#   the spoken weather functions do not need to change much
```

## Spoken Reply Segments

```python
# Goal:
# - speak sections as separate units so pauses are intentional
#
# Example response plan for Full Report:
# response_segments = [
#     {
#         "text": "Today's high is 74 degrees and the low is 58 degrees.",
#         "pause_ms": 1500,
#     },
#     {
#         "text": "Humidity is expected to stay around 62 percent this afternoon.",
#         "pause_ms": 1500,
#     },
#     {
#         "text": "There is a 20 percent chance of precipitation today.",
#         "pause_ms": 0,
#     },
# ]
#
# Possible implementation direction:
# - send each text segment through Piper
# - play one segment
# - wait for the configured pause
# - continue to the next segment
```

## Piper Reply Flow

```python
# import subprocess
#
# Goal:
# - take reply text
# - synthesize it locally with Piper
# - play it through the chosen speaker path
#
# Questions to answer while implementing:
# - will you synthesize to temporary WAV files or stream audio directly?
# - how will you keep playback from overlapping with microphone capture?
# - what will happen if TTS generation fails during a reply?
```

## End-To-End Flow

```python
# End-to-end assistant sketch:
#
# state = "idle"
#
# while running:
#     # Read the next microphone frame.
#
#     # If state == "idle":
#     #     evaluate wake phrase
#     #     if wake detected:
#     #         state = "announce_groups"
#
#     # If state == "announce_groups":
#     #     speak available groups
#     #     state = "await_group"
#
#     # If state == "await_group":
#     #     capture one utterance
#     #     recognize it with Vosk
#     #     route to a known group or retry
#
#     # If state == "announce_functions":
#     #     speak functions for the chosen group
#     #     state = "await_function"
#
#     # If state == "await_function":
#     #     capture one utterance
#     #     recognize it with Vosk
#     #     route to a known function or retry
#
#     # If state == "execute_function":
#     #     gather data
#     #     compose reply segments
#     #     state = "speak_result"
#
#     # If state == "speak_result":
#     #     play response segments with pauses
#     #     state = "idle"
```
