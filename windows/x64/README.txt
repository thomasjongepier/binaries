# VoSinth — VOSIM Formant Synthesizer

VoSinth is a polyphonic VOSIM formant synthesizer implementing the Kaegi & Tempelaars (1978) model. It generates vowel and vocal-texture sounds by summing decaying sin² pulse trains, each tuned to a specific formant frequency. Unlike subtractive synthesis, the spectral shape is built directly from the oscillator parameters — no filters required for basic phoneme shaping.

---

## How it works

Each of the four oscillators produces a burst of **N sin² pulses** per pitch period, where:
- **FREQ** sets the formant centre frequency (the pulse duration τ = 1/FREQ)
- **AMP** sets the peak amplitude of the first pulse
- **N** controls how many pulses fit per pitch period (more pulses = wider formant)
- **Q** controls inter-pulse amplitude decay — higher Q narrows the formant bandwidth

Four oscillators cover the four main vocal formants (F1–F4). Preset vowel shapes (/ee/, /ey/, /ah/, /o/, /oo/) and a nasal (/mm/) are included.

---

## Controls

| Control | Description |
|---|---|
| **F1–F4 FREQ** | Formant frequency per oscillator (80–10000 Hz) |
| **F1–F4 AMP** | Amplitude of each formant |
| **F1–F4 N** | Pulse count fraction (affects formant width) |
| **F1–F4 Q** | Formant resonance sharpness |
| **ALT POL** | Alternates pulse polarity each period (brightens timbre) |
| **MIC GATE** | Sidechain input gain before pitch/formant detection |
| **ROOT MIN/MAX** | Pitch detection range (Hz) — set to your voice range |
| **FEEDBACK** | Mixes the previous output block into the sidechain analysis |
| **FQ RAND** | Per-note random spread on formant frequencies (0 = none, 0.5 = ±50%) |
| **N RAND** | Per-note random spread on pulse count |
| **RND SPEED** | How quickly random offsets wander during a held note (0 = frozen at note-on) |

CC assignments for FREQ and N per oscillator are configurable — click the badge next to each slider to MIDI-learn or enter a CC number manually.

---

## Mic / sidechain tracking

VoSinth can track the formants and pitch of a live voice input:

- **HOLD MIC** — hold to analyse the sidechain input in real time; releases on mouse-up and stores the last detected formants in the **CAPT MIC** preset slot
- **LOCK MIC** — toggle to continuously drive formants from the sidechain
- **CAPT MIC** preset button — recalls the last captured formant values

When active, the plugin detects pitch (used as the gate — unpitched/quiet signals do not drive formants) and tracks F1–F4 via band-limited peak detection with confidence smoothing.

---

## Setting up in Reaper

VoSinth needs a **MIDI instrument track** plus an optional **sidechain track** for mic tracking.

### Basic (synthesis only)

1. Create a new track → Insert virtual instrument → select VoSinth VST3
2. Arm the track for MIDI, or draw MIDI notes in the piano roll
3. Use the on-screen keyboard or a MIDI controller to play

### With mic / sidechain tracking

1. Create a second track for your microphone input
2. On the VoSinth track, open the FX chain and enable the sidechain input pin (channels 3/4 in Reaper's routing matrix)
3. On the mic track, open routing and add a send to the VoSinth track routed to **channels 3/4** (the sidechain bus)
4. Press **HOLD MIC** or **LOCK MIC** — the FFT display will show the mic spectrum in yellow and the synth output in cyan
5. Adjust **ROOT MIN/MAX** to match your voice's pitch range; adjust **MIC GATE** if detection is unstable

> **Ableton Live**: use an Audio Effect Rack with the sidechain input routed to a chain.  
> **Logic Pro**: use the sidechain selector on the instrument channel strip.

---

## Randomness

VoSinth draws independent random offsets for each oscillator at every **note-on**. With two notes held simultaneously, all eight formant slots (4 oscillators × 2 voices) get distinct values, giving each repeated note or chord voicing a slightly different spectral character.

- **FQ RAND / N RAND** set the maximum deviation as a fraction of the current parameter value
- **RND SPEED = 0**: offsets are frozen for the life of the note — each note sounds subtly unique but stable
- **RND SPEED > 0**: offsets wander from their note-on values, producing slow formant movement at lower settings and faster shimmer at higher settings

---

## Building

Built with [JUCE](https://juce.com) and CMake. Targets VST3 and AU.

```
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```
