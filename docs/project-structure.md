# Project Structure Guide

This guide explains how to organize the assistant once you start writing your own implementation. The goal is not to force a perfect architecture on day one. The goal is to give your future code a home before the project grows messy.

## Organizing Principle

Structure the project by **responsibility**, not by the order you learned it.

Good example:

- `audio/` for sound capture and playback
- `dialogue/` for state and routing
- `services/` for external data access

Less helpful example:

- `phase1/`
- `phase2/`
- `new_stuff/`

Milestone names are good for docs and notes. They are usually poor long-term code folders.

## Suggested Top-Level Layout

```text
PieAssistant/
  README.md
  docs/
  notes/
  config/
  assets/
  models/
  src/
    assistant/
      app.py
      audio/
      dialogue/
      services/
      domain/
  tests/
```

## What Each Folder Is For

### `docs/`

Keep your tutorial, architecture notes, experiment summaries, and future setup guides here.

### `notes/`

Use this for your personal build journal:

- microphone findings
- wake-word threshold experiments
- speech-recognition mistakes you observed
- weather phrasing ideas

This is especially useful in a learning project because it preserves why you changed something.

### `config/`

Put runtime configuration here rather than scattering values through code.

Examples:

- audio device names
- wake phrase threshold
- silence timeout
- selected Piper voice
- default location for weather

Prefer a human-readable format such as `yaml`, `toml`, or `json`.

### `assets/`

Use this for small project-owned assets such as:

- canned prompt audio
- short test clips
- diagrams

Avoid putting very large downloaded models here unless you deliberately want them versioned with the repo.

### `models/`

Use this for local speech assets when you want a clear place for:

- wake-word model files
- Vosk models
- Piper voices

If the models are too large for the repo, you can still treat `models/` as the expected local folder and keep the actual files ignored from version control.

### `src/assistant/`

This is the main implementation package you will create as you build.

Recommended subfolders:

- `audio/`
- `dialogue/`
- `services/`
- `domain/`

### `tests/`

Use this for automated checks if you add them later. Even if you start with only manual validation, having a `tests/` home from the beginning makes growth easier.

## Suggested File Naming

Use names that describe the job of the file, not its size or importance.

Prefer:

- `wake_word.py`
- `speech_recognition.py`
- `weather_service.py`
- `function_catalog.py`

Avoid:

- `helpers.py`
- `misc.py`
- `new_code.py`
- `test2.py`

### Naming conventions

- folders: lowercase snake_case
- Python files: lowercase snake_case
- classes: PascalCase
- functions and variables: snake_case
- constants: UPPER_SNAKE_CASE

## Suggested Internal Layout

```text
src/assistant/
  app.py
  audio/
    input.py
    playback.py
    speech_recognition.py
    text_to_speech.py
    wake_word.py
  dialogue/
    function_catalog.py
    router.py
    states.py
  services/
    weather_service.py
  domain/
    response_segments.py
    weather_models.py
```

## Why These Names Work

### `app.py`

Good place for the top-level runtime wiring once you build it. It should connect components together, not contain every behavior directly.

### `audio/input.py`

Keeps microphone capture separate from playback and speech-specific logic.

### `audio/playback.py`

Owns output-device routing and timing between spoken segments.

### `audio/wake_word.py`

A clean home for wake detection logic and threshold tuning.

### `audio/speech_recognition.py`

Keeps STT setup and utterance transcription separate from dialogue decisions.

### `audio/text_to_speech.py`

Owns reply synthesis behavior without mixing it into routing.

### `dialogue/states.py`

A clear place to define the assistant's allowed states and maybe transitions.

### `dialogue/router.py`

Routes recognized phrases according to the current state.

### `dialogue/function_catalog.py`

The single source of truth for what the assistant advertises and supports.

### `services/weather_service.py`

Keeps provider-specific fetch and normalization logic away from speech and state logic.

### `domain/`

Useful for assistant-facing data shapes such as normalized weather information and spoken response segments.

## A Few Healthy Habits

- Create a file only when you can name its responsibility clearly.
- If two files change together all the time, revisit the boundary.
- If one file starts mixing audio, routing, and weather logic, split it.
- Keep experiments out of production-named modules until they are real.
- Record tuning decisions in `notes/` so you remember what changed and why.

## Version Control Guidance

As the project grows, think about what should and should not live in git.

Often good to commit:

- docs
- configs
- small test assets
- your own code

Often worth ignoring:

- large downloaded models
- temporary audio files
- local logs
- one-off scratch recordings

## DIY-Friendly Rule Of Thumb

If you are ever unsure where a new file belongs, ask:

1. Is this about sound in or sound out?
2. Is this about deciding what the assistant means?
3. Is this about fetching or shaping outside data?
4. Is this a note, asset, model, or config?

That question set will usually point you to the right folder.
