# Build Tutorial

This tutorial is designed to help you build the assistant yourself while understanding why each part exists. It is not a line-by-line instruction manual. Each milestone explains the goal, the thinking behind it, what to build, and how to know you are ready to move on.

## Milestone 1: Hardware And OS Grounding

### What you are learning

Before speech software matters, the Pi has to behave like a stable audio appliance. You are learning how Linux exposes audio devices, how the microphone and speaker path are chosen, and why voice projects fail early when audio routing is vague.

### What you should achieve

- Raspberry Pi OS is installed and updated.
- Your USB mini microphone is visible as an input device.
- Your output path is stable and intentionally selected.
- You understand whether your speakers are powered or amplified.

### What to pay attention to

- USB microphones often appear correctly even when their gain is too low to be useful.
- The Pi may prefer HDMI audio unless you intentionally route output elsewhere.
- If you use the 3.5 mm jack, the speaker chain must provide amplification.

### Suggested verification activities

- Find the input device name that corresponds to the USB microphone.
- Find the output device name that corresponds to your intended speaker path.
- Record a short clip and play it back through the chosen output path.
- Reboot once and confirm the device names and routing still make sense.

### Comment-only examples

```bash
# Example commands you may explore on the Pi while learning the audio stack:
# arecord -l
# aplay -l
# arecord -D <input-device> -f S16_LE -r 16000 -c 1 test.wav
# aplay -D <output-device> test.wav
#
# Learning goal:
# - identify the microphone card/device
# - identify the speaker card/device
# - confirm that 16 kHz mono recording works cleanly
```

## Milestone 2: Audio Pipeline Mental Model

### What you are learning

A voice assistant is not one feature. It is a pipeline. Audio comes in continuously, a wake phrase narrows attention, speech is captured for a short window, text is recognized, a function is chosen, a spoken reply is composed, and playback finishes before the system returns to idle.

### The loop you are aiming for

1. Listen continuously.
2. Detect the wake phrase.
3. Capture the next utterance.
4. Convert the utterance into text.
5. Match the text to a dialogue state and function.
6. Generate one or more reply segments.
7. Speak the reply with pauses.
8. Return to idle.

### What to design before coding

- How long should the assistant listen after the wake phrase?
- What counts as silence?
- What phrase should trigger function execution?
- How long should pauses be between spoken weather sections?

### Comment-only example

```python
# Conceptual loop:
#
# while assistant_is_running:
#     # 1. Read a short frame from the microphone.
#     # 2. Feed it into wake-word detection while idle.
#     # 3. When wake word fires, switch to command capture mode.
#     # 4. Buffer speech until silence or timeout.
#     # 5. Hand buffered audio to speech recognition.
#     # 6. Route recognized text through the dialogue tree.
#     # 7. Convert the chosen response into one or more spoken segments.
#     # 8. Play reply audio.
#     # 9. Return to idle.
```

## Milestone 2.5: Project Structure Before Feature Growth

### What you are learning

Voice projects become hard to reason about when every experiment lands in one file. Before the assistant grows beyond a few tests, define a folder structure that reflects responsibilities rather than implementation accidents.

### What you should achieve

- You know where future implementation files will live.
- You have a naming style for modules, configs, and notes.
- You understand which assets belong in the repo and which should stay local to the Pi.

### Structure guidance

Start simple. You do not need a large framework layout on day one. You only need enough structure to separate:

- runtime code you will write
- speech models and audio assets
- configuration
- docs and experiment notes
- tests and manual validation notes

### Suggested learning-friendly layout

See [Project Structure Guide](project-structure.md) for a full example, but the short version is:

- `src/assistant/` for your implementation
- `src/assistant/services/` for weather and future integrations
- `src/assistant/audio/` for microphone, playback, and speech-adjacent code
- `src/assistant/dialogue/` for state machine and routing logic
- `config/` for device names, wake settings, and future runtime settings
- `assets/` for small checked-in assets
- `models/` for local speech models if you choose to keep them in-repo
- `notes/` for build logs, tuning notes, and hardware findings
- `tests/` for the checks you add as the project matures

### Design advice

- Use folder names for technical boundaries, not milestones.
- Prefer nouns for packages such as `audio`, `dialogue`, and `services`.
- Prefer lowercase snake_case for file names.
- Keep one main responsibility per file as long as it stays practical.

## Milestone 3: Dialogue Tree Before Features

### What you are learning

The assistant should not improvise its structure. Define the spoken menu first. This keeps the speech recognition vocabulary small, the interaction predictable, and the implementation easier to reason about.

### Initial dialogue states

- `idle`
- `announce_groups`
- `await_group`
- `announce_functions`
- `await_function`
- `execute_function`
- `speak_result`

### Initial function groups

- `Weather Reports`
- `Clock`
- `ToDo Items`
- `Reminders`
- `Timers`

### Initial weather functions

- `Full Report`
- `High and Low Temps`
- `Humidity`
- `Precipitation`

### Dialogue design guidance

Keep the early version highly constrained. Let the assistant listen for exact or near-exact phrases before you try to make it more conversational. A small grammar will teach you more than a broad one in the first version.

### Comment-only example

```python
# Spoken menu flow:
#
# User: <wake phrase>
# Assistant: "Available function groups are Weather Reports, Clock, ToDo Items, Reminders, and Timers."
# User: "Weather Reports"
# Assistant: "Weather Reports includes Full Report, High and Low Temps, Humidity, and Precipitation."
# User: "Execute function Full Report"
# Assistant: <speak weather sections with pauses>
#
# State transitions:
# idle -> announce_groups -> await_group -> announce_functions -> await_function -> execute_function -> speak_result -> idle
```

## Milestone 4: Local Speech Stack

### What you are learning

Each speech component has one job:

- `openWakeWord` detects the wake phrase.
- `Vosk` turns a short captured utterance into text.
- `Piper` turns reply text into audible speech.

Keeping those responsibilities separate makes debugging much easier.

### Why this stack fits the project

- It runs locally on a Pi 4/5.
- It avoids turning a learning project into an API integration project.
- It lets you tune each stage independently.
- It supports the menu-driven flow well.

### Design advice

- Start with an existing wake phrase model before training your own.
- Keep Vosk vocabulary constrained to menu phrases first.
- Use Piper for short, deliberate replies instead of long conversational speech.

### Things to tune

- wake-word threshold
- silence timeout
- speech capture length
- recognition vocabulary
- TTS voice and pacing

## Milestone 5: Function Catalog

### What you are learning

The assistant needs a clean internal map from spoken categories to executable behaviors. Even before you implement the behaviors, the shape of that catalog should be clear.

### What the catalog should represent

- display name
- spoken description
- list of child functions for each group
- implementation target for each leaf function
- whether a function is complete, stubbed, or placeholder-only

### Suggested first-pass structure

- Make `Weather Reports` the first fully planned group.
- Keep the other groups present but lightly stubbed.
- Treat `Full Report` as a composition function, not a separate data source.

### Comment-only example

```python
# Conceptual catalog shape:
#
# FUNCTION_GROUPS = {
#     "Weather Reports": {
#         "spoken_label": "Weather Reports",
#         "functions": {
#             "Full Report": "compose all weather sections",
#             "High and Low Temps": "read daily high and low",
#             "Humidity": "read humidity summary",
#             "Precipitation": "read precipitation summary",
#         },
#     },
#     "Clock": {
#         "spoken_label": "Clock",
#         "functions": {
#             # Placeholder ideas:
#             # "Current Time": "read the current local time"
#             # "Current Date": "read the current date"
#         },
#     },
# }
```

## Milestone 6: Weather Report Composition

### What you are learning

Raw weather API fields are not a spoken experience yet. You need a normalization layer that converts forecast data into concise spoken sections.

### What to normalize

- daily high
- daily low
- humidity value or range
- precipitation chance or expected precipitation
- location label
- forecast date context such as today or tomorrow

### Spoken design goal

The assistant should sound structured, not rushed. `Full Report` should present:

1. High and low temperatures
2. A pause
3. Humidity
4. A pause
5. Precipitation

### Composition guidance

- Keep each section short enough to understand on first listen.
- Use stable section names so the assistant feels consistent.
- Avoid dumping every weather field the API offers.
- Treat pauses as part of the response, not an afterthought.

### Comment-only example

```python
# Example full-report composition idea:
#
# high_low_text = "Today's high is 74 degrees and the low is 58 degrees."
# humidity_text = "Humidity is expected to stay around 62 percent this afternoon."
# precipitation_text = "There is a 20 percent chance of precipitation today."
#
# response_segments = [
#     {"text": high_low_text, "pause_ms": 1500},
#     {"text": humidity_text, "pause_ms": 1500},
#     {"text": precipitation_text, "pause_ms": 0},
# ]
#
# Learning goal:
# - each segment becomes one TTS unit
# - the pause is explicit, predictable, and testable
```

## Milestone 7: Reliability And Polish

### What you are learning

The difference between a demo and a usable assistant is usually error handling. Your first version should know how to recover when it hears nothing, hears noise, or hears a phrase outside the known menu.

### Behaviors worth adding

- retry prompt when recognition is uncertain
- timeout when no command follows the wake phrase
- reset to idle after completed playback
- optional confirmation for destructive future commands
- logging for recognized phrases and state transitions

### Good failure messages

- "I didn't catch that. Please say a function group."
- "I heard Weather Reports. Say a function name."
- "That function is not available yet."

### What not to optimize too early

- natural conversation
- multiple wake phrases
- multi-user recognition
- complex reminders database
- background music ducking

## Build Order Recommendation

Implement in this order:

1. Stable audio input and output
2. Manual playback of a canned spoken reply
3. Menu flow driven by typed test input
4. Speech-to-text for menu choices
5. Wake phrase activation
6. Weather report composition
7. Recovery behavior and tuning

That order teaches faster because it isolates one unknown at a time.
