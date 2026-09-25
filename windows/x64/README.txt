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
| **MIC GATE** | Mic input gain before pitch/formant detection (0–40 dB) |
| **ROOT MIN/MAX** | Pitch detection range (Hz) — set to your voice range |
| **DYNAMICS** | How much the synth level follows your voice's loudness (0 = fixed, 1 = fully) |
| **MONITOR** | A/B switch: hear the synth or your dry mic, to compare |
| **CALIBRATE…** | Guided voice calibration and training session (see below) |
| **FEEDBACK** | Mixes the synth output back into the mic analysis — an effect, not a stabiliser; keep at 0 for the most faithful tracking |
| **FQ RAND** | Per-note random spread on formant frequencies (0 = none, 0.5 = ±50%) |
| **N RAND** | Per-note random spread on pulse count |
| **RND SPEED** | How quickly random offsets wander during a held note (0 = frozen at note-on) |

CC assignments for FREQ and N per oscillator are configurable — click the badge next to each slider to MIDI-learn or enter a CC number manually.

---

## Mic tracking

VoSinth can follow a live voice: your formants drive the four oscillators while you play notes on the keyboard.

- **HOLD MIC** — hold to analyse the mic in real time; on release the detected formants are saved as a new "Mic Capture" OSC preset
- **LOCK MIC** — toggle to continuously drive the formants from the mic while a pitched voice is detected

How it works:

- Pitch is detected with **YIN**; only voiced, pitched input drives the formants
- Formants are found with **LPC** (linear prediction), which gives each formant's frequency *and* bandwidth, and they are tracked smoothly through vowel changes
- Each oscillator gets its formant's frequency, level and **Q = F / (π·B)** from the measured bandwidth; levels are compensated for the oscillators' own gain so the synth keeps your voice's spectral balance
- While the mic is driving, F2 and F4 are sign-inverted (as in a parallel formant synthesiser), so the valleys between formants fill in naturally
- A slow closed loop compares the input and output spectra and trims each formant's level toward your voice
- The FFT display shows the mic in yellow and the synth in cyan, plus a **MATCH** score (100% = same formant balance) and the per-formant difference in dB

### Calibration and training

Open the mic settings and press **CALIBRATE…** for a guided session (about 40 s): stay silent, glide through your pitch range, then sing *ah*, *eh*, *ee*, *oh*, *oo* and hum *mm*. VoSinth then:

- sets the noise gate and **ROOT MIN/MAX** from your voice
- narrows the formant search to your own vowel space
- runs a short optimisation per vowel (a few seconds in the background) that searches the oscillator settings best matching your recording, and applies the learned corrections live

Toggle **USE TRAINING** to compare with and without the learned corrections. Calibration results are saved with the project.

### Recommended mic setup

- **Use headphones** — speakers leak the synth into the mic
- **Turn off Windows audio enhancements** for the mic (Settings → System → Sound → your microphone → Audio enhancements → Off), or use WASAPI Exclusive / ASIO; noise suppression and automatic gain control distort the analysis
- An external or USB mic tracks more faithfully than a built-in laptop mic
- Any buffer size works (128 samples is fine)

---

## Setting up in Reaper

VoSinth needs a **MIDI instrument track** plus a **mic track** for mic tracking.

### Basic (synthesis only)

1. Create a new track → Insert virtual instrument → select VoSinth VST3
2. Arm the track for MIDI, or draw MIDI notes in the piano roll
3. Use the on-screen keyboard or a MIDI controller to play

### With mic tracking

Reaper does not take MIDI and audio input on the same instrument track, so use two tracks:

1. **Track 1** — VoSinth, receiving MIDI
2. **Track 2** — your microphone: set its input to the mic, arm it with monitoring on, and **disable its master send** (so the dry mic does not reach the mix)
3. On Track 2, add a send to Track 1 on **channels 1/2** (VoSinth's main input)
4. Press **HOLD MIC** or **LOCK MIC** — the FFT display shows the mic spectrum in yellow
5. Run **CALIBRATE…** once, or set **ROOT MIN/MAX** to your range and adjust **MIC GATE** if detection is unstable

To use the sidechain bus instead (channels 3/4), switch **ROUTING** in the mic settings to *Sidechain*.

---

## Randomness

VoSinth draws independent random offsets for each oscillator at every **note-on**. With two notes held simultaneously, all eight formant slots (4 oscillators × 2 voices) get distinct values, giving each repeated note or chord voicing a slightly different spectral character.

- **FQ RAND / N RAND** set the maximum deviation as a fraction of the current parameter value
- **RND SPEED = 0**: offsets are frozen for the life of the note — each note sounds subtly unique but stable
- **RND SPEED > 0**: offsets wander from their note-on values, producing slow formant movement at lower settings and faster shimmer at higher settings

---

## Building

Built with [JUCE](https://juce.com) from a Projucer project (`VoSinth.jucer`) and Visual Studio 2022 (Release | x64). This binary is the Windows x64 VST3; copy `VoSinth.vst3` to `C:\Program Files\Common Files\VST3` and rescan plugins in your DAW.
