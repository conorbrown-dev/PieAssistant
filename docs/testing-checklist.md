# Testing Checklist

Use this checklist as you build. It is written for manual validation on the Raspberry Pi.

## Audio Foundation

- Confirm the USB microphone appears consistently after reboot.
- Confirm the intended output device appears consistently after reboot.
- Confirm you can record clean mono speech at the sample rate you chose.
- Confirm playback reaches the correct speakers, not HDMI by accident.
- Confirm the 3.5 mm output path is feeding powered or amplified speakers.

## Wake Phrase

- Confirm the wake phrase advances the assistant from `idle` to menu mode.
- Confirm room noise alone does not frequently trigger wake.
- Confirm normal speaking volume from expected distance is recognized reliably.

## Group Selection

- Confirm the assistant announces all five function groups clearly.
- Confirm `Weather Reports` is recognized as a valid group.
- Confirm unknown group phrases trigger a retry prompt instead of a crash.

## Function Selection

- Confirm the assistant announces the four weather functions clearly.
- Confirm `Execute function Full Report` routes to the correct behavior.
- Confirm the other three weather functions can be routed individually.
- Confirm unsupported or not-yet-built functions produce a graceful response.

## Weather Response

- Confirm `Full Report` contains:
  - high and low temperatures
  - humidity
  - precipitation
- Confirm the sections are spoken in the intended order.
- Confirm pauses between weather sections feel intentional and audible.
- Confirm the assistant returns to `idle` after the full report finishes.

## Recovery Behavior

- Confirm silence after wake causes a timeout and reset.
- Confirm low-confidence or unrecognized speech causes a retry prompt.
- Confirm the assistant recovers cleanly after an STT or TTS failure.
- Confirm repeated failed inputs do not leave the assistant in a broken state.

## Suggested Build Milestone Gates

Only move to the next milestone when the current one is stable:

1. Audio device selection survives reboot.
2. One canned spoken reply plays successfully.
3. Typed input can drive the dialogue tree.
4. STT can reliably choose groups and functions.
5. Wake phrase can start the interaction reliably.
6. Weather report sections are composed and spoken with pauses.
7. Retry and timeout behavior feel calm and predictable.
