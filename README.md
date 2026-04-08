
Demo:
https://iusmusic.github.io/fivetrack/

# I/US 5 Track Recorder

A browser-based 5-track audio recorder and mixer built as a single HTML application with the Web Audio API.

This app is designed for fast multitrack sketching directly in the browser: choose an input device, enable audio, record into one of five tracks, shape each track with basic effects, monitor the master bus, and export either a stereo mix or split mono files.

## What the app does

- Records **stereo audio** into up to **5 tracks**
- Uses an **AudioWorklet-based recorder** for input capture
- Provides **per-track mixing controls**:
  - volume
  - pan
  - delay mix
  - delay time
  - feedback
  - reverb mix
- Provides **master bus controls**:
  - input gain
  - master gain
  - compressor threshold
  - compressor ratio
  - makeup gain
- Supports **input monitoring** on or off
- Displays **per-track meters**, a **master meter**, and a global status bar
- Exports either:
  - a **stereo WAV mix**, or
  - **two mono WAV files** for left/right channels
- Supports **Normal Export** and **Loud Export**
- Applies **peak normalization on export**

## Tech stack

This application is implemented as a single self-contained HTML file and uses:

- **HTML/CSS** for layout and styling
- **Vanilla JavaScript** for state, UI wiring, and export logic
- **Web Audio API** for audio routing, effects, monitoring, metering, and offline rendering
- **AudioWorklet** for real-time stereo recording capture
- **OfflineAudioContext** for final export rendering

No external framework or build system is required.

## Core architecture

### 1. Input stage

After audio is enabled, the app:

1. creates an `AudioContext`
2. requests microphone or device input with `getUserMedia`
3. creates a media source from the selected input device
4. routes input through an **input gain** stage
5. optionally routes monitored audio to the master path when monitoring is enabled
6. sends audio into an **AudioWorkletNode** for stereo capture

### 2. Recorder stage

The recorder worklet captures **left** and **right** channel sample blocks and posts them back to the main thread. Each active recording track accumulates stereo chunks into internal buffers.

When recording stops, the app assembles those chunks into an `AudioBuffer`.

### 3. Track playback stage

Each track builds its own playback graph using Web Audio nodes. A track source is routed through per-track processing including:

- gain
- stereo panning
- feedback delay path
- convolution reverb path

Each track can be:

- recorded
- stopped
- played
- looped
- cleared
- selected for export targeting

### 4. Master stage

The mix bus is routed through a master chain that includes:

- master gain
- compressor
- makeup gain
- limiter

A master analyser node drives the master meter.

### 5. Export stage

Exports are rendered offline using `OfflineAudioContext` rather than recorded from live playback. This makes export deterministic and allows the app to apply its master processing consistently.

The export renderer:

1. gathers either selected tracks or all populated tracks
2. reconstructs the track processing graph in an offline context
3. applies the master compressor, makeup gain, and limiter
4. renders the mix
5. peak-normalizes the result
6. downloads either:
   - one stereo WAV, or
   - two mono WAV files

## UI overview

## Top bar

### Audio initialization
- **Enable Audio**: initializes audio, loads devices, and prepares the recorder and mixer
- **Input Device**: selects the capture source
- **Sample Rate**: shows the active audio context sample rate
- **Monitor On / Off**: toggles live monitoring of the input signal

### Master controls
- **In**: input gain before recording and monitoring
- **Vol**: master gain
- **Comp**: compressor threshold
- **Ratio**: compressor ratio
- **Makeup**: makeup gain after compression
- **Master meter**: live master level display
- **Status bar**: current system message or action feedback

### Global transport/export controls
- **Play All**: starts playback of all populated tracks
- **Stop**: stops recording and playback across tracks
- **Clear All**: clears all tracks
- **Export Channel Mode**:
  - `Export Stereo WAV`
  - `Export Two Mono WAVs`
- **Export Loudness Mode**:
  - `Normal Export`
  - `Loud Export`
- **Export**: renders and downloads the mix

## Per-track UI

Each of the five tracks contains:

- track ID and status
- ready/record/loop indicators
- selection toggle for export targeting
- waveform display
- track level meter
- **PLAY** button
- **REC**, **STOP**, **LOOP**, and **CLR** buttons
- per-track FX controls:
  - Volume
  - Pan
  - Delay Mix
  - Delay Time
  - Feedback
  - Reverb Mix

## How to use the app

### Basic workflow

1. Open the HTML file in a modern desktop browser.
2. Click **Enable Audio**.
3. Allow microphone/audio device permissions when prompted.
4. Choose the desired **Input Device**.
5. Set **Monitor On** only if you want to hear live input through the app.
6. Adjust **In** to set the recording input gain.
7. On a track, click **REC** to begin recording.
8. Click **STOP** to end recording.
9. Click **PLAY** to audition the track.
10. Adjust track FX and track volume as needed.
11. Repeat on other tracks.
12. Use **Play All** to audition the multitrack mix.
13. Choose an export mode and click **Export**.

### Recording notes

- Only one track records at a time.
- Starting a new recording track stops any other active recording track.
- Tracks remain available for playback and export until cleared.
- The waveform area is used for visual feedback rather than destructive editing.

### Export behavior

If one or more tracks are selected using their **SELECT** toggle, export will target only those selected tracks.

If no tracks are selected, export will include all populated tracks.

## Export modes

### Stereo WAV

Exports a single stereo file containing the rendered mix.

### Two Mono WAVs

Exports two separate mono files:

- left channel
- right channel

This is useful when you need split stems for external processing or downstream routing.

### Normal Export

Uses the configured master chain with standard dynamics settings.

### Loud Export

Uses a stronger master rendering profile by increasing effective output level and tightening limiting so the exported file comes out stronger than the default render.

## Signal flow

### Live path

`Input Device -> Input Gain -> (optional monitor path) -> AudioWorklet recorder -> track buffers`

### Playback/mix path

`Track BufferSource -> Track Gain -> Delay/Reverb sends -> Pan -> Mix Bus -> Master Gain -> Compressor -> Makeup Gain -> Limiter -> Output`

### Export path

`Offline Track Sources -> Offline Track FX -> Offline Mix Bus -> Offline Master Chain -> Peak Normalization -> WAV encoding/download`

## Loudness and normalization

The app includes two different concepts that are easy to confuse:

### Makeup gain

Makeup gain increases level after compression and before limiting on the master chain.

### Peak normalization on export

After offline rendering, the app scans the rendered buffer and scales it so the peak reaches the target ceiling. This helps produce consistently strong exports and avoids unnecessarily quiet output files.

### Loud Export mode

Loud Export applies a more aggressive export profile than Normal Export. This is intended for users who want a stronger rendered file without needing to manually push all gain stages.

## Browser requirements

Recommended browser support:

- recent **Chrome**
- recent **Edge**
- recent **Safari** with Web Audio support
- recent **Firefox** with AudioWorklet support

For best results, use a browser that fully supports:

- `AudioWorklet`
- `MediaDevices.getUserMedia`
- `OfflineAudioContext`
- `StereoPannerNode`

## Running locally

Because this app is a plain HTML file, the simplest way to use it is to open it directly in a browser.

If your browser applies stricter local file restrictions, serve it from a small local static server instead.

Example options:

- Python: `python -m http.server`
- any simple local static server

Then open the served HTML file in the browser.

## Project structure

The current implementation is intentionally minimal.

- `Dindexlogo_updated.html` — full app, UI, audio engine, recorder worklet bootstrap, export logic
- `README.md` — this document

## Known limitations

- This is not a DAW; it is a lightweight browser multitrack recorder.
- There is no non-destructive timeline editing, trimming, slicing, or clip movement.
- There is no persistent project save/load format.
- Effects are intentionally basic and optimized for immediacy.
- Browser audio permission behavior differs by platform and browser.
- Monitoring may produce feedback if speakers and microphone are both active.
- Export is WAV-focused and does not currently provide MP3/AAC encoding.

## Troubleshooting

### No sound or no input device
- Click **Enable Audio** first.
- Verify browser microphone permission is granted.
- Re-check the selected input device.
- Confirm the device is not already locked by another application.

### Recording is too quiet
- Increase **In**.
- Increase track **Volume** only for playback/mix balance.
- Use **Makeup** and/or **Loud Export** for stronger exported output.

### Distortion or clipping
- Lower **In** if the source is too hot.
- Reduce track volume and/or **Vol**.
- Use the compressor and limiter more conservatively.

### Feedback when monitoring
- Switch **Monitor Off**.
- Use headphones instead of speakers.

### Export seems different from live playback
- Export is rendered offline and may sound tighter because the offline master chain and peak normalization are part of the export path.
- Verify whether **Normal Export** or **Loud Export** is selected.

## Future extension ideas

- project save/load
- per-track mute/solo
- waveform trimming and region editing
- overdub/punch-in behavior
- drag-and-drop import of audio clips
- stem export per track
- LUFS metering and target presets
- keyboard shortcuts


This repository is source-available, not open source.

It is licensed under the I/US Source-Available License 1.0. You may review the source code and use it for limited private internal evaluation, but redistribution, public derivative distribution, and commercial use are not permitted without prior written permission.

Copyright
Copyright (c) 2026 Pezhman Farhangi

Contact
For licensing requests, commercial rights, redistribution requests, or permission to use protected brand assets, prior written permission must be obtained from I/US Music.
