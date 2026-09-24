# Detailed Documentation — GNU Radio DFR / RF Interference Experiments

## 1. Project Overview

This project consists of three GNU Radio Companion flowgraphs:

```text
JammerTX.grc
jammerRX.grc
JammerRXTX.grc
```

The flowgraphs form a related experimental set for studying an NBFM communication path together with a configurable complex Gaussian-noise path. The designs demonstrate how an audio source can be converted into an NBFM complex signal, how an RF receiver can capture and demodulate an NBFM signal, and how signal/noise paths can be observed independently and in combination.

The project is implemented using GNU Radio's UHD interface for USRP hardware. QT GUI frequency sinks are used throughout the experiments to observe spectral behavior.

> **Important scope note:** The documentation below describes what is implemented in the supplied GRC files. It does not claim experimental measurements that are not stored in the flowgraphs. Actual received audio quality, interference range, BER, SNR improvement/degradation, or RF power measurements cannot be determined from the GRC files alone.

---

# 2. Project Architecture

The three flowgraphs can be viewed as three related experimental configurations.

### Flowgraph A — `JammerTX.grc`

```text
                         ┌──► Audio Sink
                         │
WAV Audio ──► NBFM TX ───┼──► Frequency Sink ──► USRP Sink
                         │
                         └─────────────────────► RF signal path

Gaussian Noise ──► LPF ──► Frequency Sink ──► USRP Sink
```

### Flowgraph B — `jammerRX.grc`

```text
USRP Source
     │
     ▼
Low-Pass FIR Filter
     │
     ▼
NBFM Receiver
     │
     ├──► Audio Sink
     │
     └──► Frequency Sink
```

### Flowgraph C — `JammerRXTX.grc`

```text
WAV Audio
    │
    ▼
 NBFM TX ───────────────► Frequency Sink
    │
    ├──────────────► Signal USRP Sink
    │
    ▼
   Adder ◄──────── Gaussian Noise
    │                    │
    ├──► Frequency Sink  ├──► Noise Frequency Sink
    │                    │
    ▼                    └──► Noise USRP Sink
 Low-Pass Filter
    │
    ▼
 NBFM RX
    │
    ├──► Audio Sink
    │
    └──► Frequency Sink

                    ^
                    |
               USRP Source
```

---

# 3. Software and Hardware

## Software

The flowgraphs use:

- GNU Radio Companion
- GNU Radio UHD blocks
- GNU Radio Analog blocks
- GNU Radio Blocks
- GNU Radio QT GUI
- Python-generated GNU Radio runtime

## Hardware Interface

The RF portions use UHD USRP Source/Sink blocks.

The supplied files contain specific USRP device identifiers. These identifiers are machine-specific and may need to be changed when moving the flowgraphs to another computer or USRP setup.

---

# 4. Global Parameters

## 4.1 Sample Rate

All three flowgraphs use:

```text
Sample Rate = 240000 samples/s
```

or:

```text
240 kS/s
```

This sample rate is used as the quadrature/sample-processing rate throughout the RF and NBFM paths.

---

# 5. Flowgraph 1 — `jammerRX.grc`

## 5.1 Purpose

`jammerRX.grc` is the receiver-side flowgraph. It captures complex samples from a USRP, filters the received signal, performs NBFM demodulation, and outputs recovered audio.

The main signal chain is:

```text
UHD USRP Source
       │
       ▼
Low-Pass FIR Filter
       │
       ▼
NBFM Receiver
       │
       ├────────► Audio Sink
       │
       └────────► QT GUI Frequency Sink
```

---

## 5.2 Parameters

| Parameter | Default | Range / Configuration |
|---|---:|---|
| Center Frequency | 433 MHz | Fixed variable |
| Sample Rate | 240 kS/s | Fixed variable |
| Receiver Gain | 20 | 0–40 |
| Filter Cutoff | 8 kHz | 5–20 kHz |
| Transition Width | 2 kHz | 1–5 kHz |
| Audio Rate | 48 kS/s | Fixed |
| NBFM Max Deviation | 5 kHz | Block parameter |
| NBFM Tau | 75 µs | Block parameter |

---

# 6. Blocks in `jammerRX.grc`

## 6.1 `variable` — `center_freq`

```text
Value = 433000000
```

This defines the RF center frequency used by the USRP Source and the frequency sink.

---

## 6.2 `variable` — `samp_rate`

```text
Value = 240000
```

This defines the complex sample rate used by the RF processing chain.

---

## 6.3 `QT GUI Range` — `receive_gain`

The receiver gain is exposed as a GUI control.

```text
Minimum = 0
Maximum = 40
Step    = 1
Default = 20
```

It controls the gain parameter of the USRP receiver.

---

## 6.4 `QT GUI Range` — `cutoff_filter`

This controls the cutoff frequency of the receive low-pass filter.

```text
Minimum = 5000 Hz
Maximum = 20000 Hz
Step    = 500 Hz
Default = 8000 Hz
```

The purpose is to limit the received complex signal before NBFM demodulation.

---

## 6.5 `QT GUI Range` — `transition_width`

This controls the FIR filter transition width.

```text
Minimum = 1000 Hz
Maximum = 5000 Hz
Step    = 1000 Hz
Default = 2000 Hz
```

---

## 6.6 `UHD USRP Source`

This is the RF input block.

The supplied flowgraph configures:

```text
Antenna      = RX2
Center Freq  = center_freq
Gain         = receive_gain
Sample Rate  = samp_rate
```

The flowgraph also contains a specific USRP serial identifier. This should be treated as part of the original hardware setup rather than a portable configuration.

The block outputs complex baseband samples.

---

## 6.7 `Low Pass Filter`

The received complex signal is passed through a FIR low-pass filter.

Configuration:

```text
Type           = interp_fir_filter_ccf
Interpolation  = 1
Decimation     = 1
Gain           = 1
Window         = Hamming
Beta           = 6.76
Cutoff         = cutoff_filter
Transition     = transition_width
Sample Rate    = samp_rate
```

Its role is to restrict the received signal bandwidth before NBFM demodulation.

---

## 6.8 `NBFM Receiver`

The NBFM receiver is configured with:

```text
Quadrature Rate = samp_rate
Audio Rate      = 48000
Maximum Dev.    = 5 kHz
Tau             = 75 µs
```

It converts the filtered complex NBFM signal into a real audio stream.

---

## 6.9 `Audio Sink`

The recovered audio is sent to the computer audio device.

```text
Sample Rate = 48000 samples/s
Inputs      = 1
```

---

## 6.10 `QT GUI Frequency Sink`

The demodulated signal is displayed in the frequency domain.

The supplied configuration uses:

```text
FFT Size       = 1024
Bandwidth      = samp_rate
Center Freq    = center_freq
Window         = Blackman-Harris
Update Time    = 0.10 s
Y-axis         = -140 to +10 dB
```

The sink is labelled:

```text
de-modulated
```

---

# 7. Flowgraph 2 — `JammerTX.grc`

## 7.1 Purpose

`JammerTX.grc` implements a transmit-side experimental configuration. It contains:

1. A WAV audio source
2. NBFM modulation
3. A configurable Gaussian complex-noise source
4. Noise filtering
5. Two USRP transmit paths
6. Frequency-domain monitoring
7. Local audio monitoring
8. A receive and demodulation path for monitoring

The signal and noise are therefore represented as separate RF processing paths in this flowgraph.

---

# 8. Parameters of `JammerTX.grc`

| Parameter | Default | Range / Configuration |
|---|---:|---|
| Center Frequency | 435 MHz | Fixed variable |
| Sample Rate | 240 kS/s | Fixed |
| Jam Power | 0 | 0–10, step 0.1 |
| Noise Gain | 0 | 0–1, step 0.1 |
| Signal Gain | 0 | 0–50, step 1 |
| Receiver Gain | 20 | 0–40, step 1 |
| Noise LPF Cutoff | 20 kHz | Fixed |
| Noise LPF Transition | 2 kHz | Fixed |
| Signal LPF Cutoff | GUI-controlled | 2–20 kHz |
| Signal LPF Transition | GUI-controlled | 1–5 kHz |

---

# 9. Blocks in `JammerTX.grc`

## 9.1 WAV File Source

The flowgraph uses a WAV file as its audio source.

The supplied path is:

```text
D:\God Mode Begins - Masstamilan.MY.wav
```

The source is configured as:

```text
Channels = 1
Repeat   = True
```

The file path is machine-specific and must be changed when running the flowgraph on another computer.

---

## 9.2 NBFM Transmitter

The audio stream is converted into a complex NBFM signal.

Configuration:

```text
Audio Rate      = 48000
Quadrature Rate = 240000
Maximum Dev.    = 5 kHz
Tau             = 75 µs
```

This creates the complex baseband signal used by the USRP transmitter.

---

## 9.3 Gaussian Noise Source

The noise source is:

```text
Type       = Complex
Noise Type = Gaussian
Amplitude  = jam_power
Seed       = 0
```

The `jam_power` GUI control determines the amplitude of the generated complex noise.

---

## 9.4 Noise Low-Pass Filter

The generated noise passes through a FIR low-pass filter.

```text
Cutoff Frequency = 20 kHz
Transition Width = 2 kHz
Window           = Hamming
Gain             = 1
```

This limits the spectral content of the noise before it reaches the noise monitoring/transmit path.

---

## 9.5 Frequency Sinks

The flowgraph contains three relevant spectral monitoring points:

### Original NBFM Signal

Connected directly to the output of the NBFM transmitter.

Label:

```text
original
```

### Noise

Connected to the filtered noise signal.

Label:

```text
noise freq
```

### Demodulated Signal

Connected to the NBFM receiver output.

Label:

```text
de-modulated
```

These displays allow the experiment to compare the spectral characteristics of the signal, noise, and recovered output.

---

## 9.6 USRP Sinks

Two UHD USRP Sink blocks are present.

The first receives the NBFM signal and uses:

```text
Center Frequency = center_freq
Gain             = signal_gain
Sample Rate      = samp_rate
Antenna          = TX/RX
```

The second receives the filtered noise and uses:

```text
Center Frequency = 432 MHz
Gain             = noise_gain
Sample Rate      = samp_rate
Antenna          = TX/RX
```

The exact device serial identifier is embedded in the original flowgraph.

---

## 9.7 USRP Source

A UHD USRP Source is also present for the receive/monitoring branch.

It uses:

```text
Antenna      = RX2
Center Freq  = center_freq
Gain         = receive_gain
Sample Rate  = samp_rate
```

The received samples are passed through the receive low-pass filter and then the NBFM receiver.

---

# 10. Flowgraph 3 — `JammerRXTX.grc`

## 10.1 Purpose

`JammerRXTX.grc` is the most complete of the three supplied flowgraphs. It combines:

- Audio source
- NBFM transmission
- Gaussian noise generation
- Signal/noise addition
- Separate signal and noise USRP transmit paths
- USRP reception
- Low-pass filtering
- NBFM demodulation
- Audio output
- Multiple spectrum displays

The flowgraph is therefore useful for observing the relationship between the original FM signal, generated noise, combined signal, and recovered demodulated output.

---

# 11. Parameters of `JammerRXTX.grc`

| Parameter | Default | Range |
|---|---:|---:|
| Center Frequency | 915 MHz | Fixed |
| Sample Rate | 240 kS/s | Fixed |
| Cutoff Frequency | 8 kHz | 2–20 kHz |
| Transition Width | 2 kHz | 1–5 kHz |
| Jam Power | 0 | 0–10 |
| Noise Gain | 0 | 0–1 |
| Signal Gain | 0 | 0–40 |
| Receiver Gain | 0 | 0–40 |

---

# 12. Blocks in `JammerRXTX.grc`

## 12.1 WAV File Source

The audio source is:

```text
I:\SKILLS\SDDR RADIO\PROJECT\bassbasic-new-edm-music-beet-mr-sandeep-rock-141616 (1).wav
```

Configuration:

```text
Channels = 1
Repeat   = True
```

The absolute path is specific to the original machine.

---

## 12.2 NBFM Transmitter

The WAV audio is converted to complex NBFM.

```text
Audio Rate      = 48000
Quadrature Rate = 240000
Maximum Dev.    = 5 kHz
Tau             = 75 µs
```

The NBFM output is connected to:

- Original signal frequency sink
- Signal USRP sink
- Two-input adder

---

## 12.3 Gaussian Noise Source

The noise generator produces complex Gaussian noise.

```text
Type       = Complex
Noise Type = Gaussian
Amplitude  = jam_power
Seed       = 0
```

The noise output is connected to:

- Two-input adder
- Noise frequency sink
- Noise USRP sink

---

## 12.4 Two-Input Adder

The `blocks_add_xx` block has:

```text
Number of inputs = 2
Data type        = Complex
Vector length    = 1
```

It receives:

```text
Input 0 = NBFM signal
Input 1 = Gaussian noise
```

and produces the combined complex signal.

This is the key baseband combination stage of the experiment.

---

## 12.5 Combined-Signal Frequency Sink

The combined output of the adder is connected to a QT GUI frequency sink labelled:

```text
added noise
```

This display provides a spectral view of the combined signal/noise stream.

---

## 12.6 USRP Signal Sink

The NBFM signal is sent to a UHD USRP Sink with:

```text
Center Frequency = center_freq
Gain             = signal_gain
Sample Rate      = samp_rate
Antenna          = TX/RX
```

---

## 12.7 USRP Noise Sink

The Gaussian noise is sent through a separate UHD USRP Sink.

```text
Center Frequency = center_freq
Gain             = noise_gain
Sample Rate      = samp_rate
Antenna          = TX/RX
```

The supplied file uses a separate USRP device identifier for this flowgraph.

---

## 12.8 UHD USRP Source

The receive chain starts with a UHD USRP Source configured with:

```text
Center Frequency = center_freq
Gain             = receive_gain
Sample Rate      = samp_rate
Antenna          = RX2
```

The received complex samples are passed to the low-pass filter.

---

## 12.9 Receive Low-Pass Filter

The receive filter uses:

```text
Cutoff Frequency = cutoff_filter
Transition Width = transition_width
Window           = Hamming
Beta             = 6.76
Gain             = 1
Interpolation    = 1
Decimation       = 1
```

Its purpose is to limit the received bandwidth before NBFM demodulation.

---

## 12.10 NBFM Receiver

The receiver uses:

```text
Quadrature Rate = samp_rate
Audio Rate      = 48000
Maximum Dev.    = 5 kHz
Tau             = 75 µs
```

The output is a recovered audio stream.

---

## 12.11 Audio Sink

The recovered audio is sent to the system audio device at:

```text
48 kS/s
```

---

# 13. QT GUI Frequency Sinks

The project uses frequency-domain visualization extensively.

The configured FFT size is generally:

```text
1024
```

and the window type is:

```text
Blackman-Harris
```

The displays are used to observe different stages.

| Display | Flowgraph | Purpose |
|---|---|---|
| `original` | JammerTX / JammerRXTX | Observe the NBFM signal |
| `noise freq` | JammerTX / JammerRXTX | Observe generated noise |
| `added noise` | JammerRXTX | Observe combined signal/noise |
| `de-modulated` | Receiver paths | Observe recovered audio spectrum |

---

# 14. Complete Signal Processing

The conceptual processing chain is:

```text
                 AUDIO SOURCE
                      │
                      ▼
                WAV File Source
                      │
                      ▼
                 NBFM TX
                      │
                      ├──────────────► Signal Spectrum
                      │
                      ▼
                 Signal Path
                      │
                      └──────────────► USRP TX


        Gaussian Noise Generator
                  │
                  ▼
             Noise Filter
                  │
                  ├──────────────► Noise Spectrum
                  │
                  └──────────────► USRP TX


        Signal + Noise Baseband
                  │
                  ▼
             Adder / Mixer
                  │
                  ▼
          Combined Spectrum
```

The receive side is:

```text
                RF / USRP
                    │
                    ▼
             UHD USRP Source
                    │
                    ▼
             Low-Pass Filter
                    │
                    ▼
              NBFM Receiver
                    │
             ┌──────┴──────┐
             ▼             ▼
        Audio Sink     Spectrum Sink
```

---

# 15. Role of the Main GNU Radio Blocks

| Block | Function |
|---|---|
| `blocks_wavfile_source` | Reads the source audio from a WAV file |
| `analog_nbfm_tx` | Converts audio into complex NBFM |
| `analog_noise_source_x` | Generates complex Gaussian noise |
| `blocks_add_xx` | Adds the signal and noise streams |
| `uhd_usrp_sink` | Sends complex samples to USRP hardware |
| `uhd_usrp_source` | Receives complex samples from USRP hardware |
| `low_pass_filter` | Restricts signal bandwidth |
| `analog_nbfm_rx` | Demodulates NBFM into audio |
| `audio_sink` | Plays recovered/local audio |
| `qtgui_freq_sink_x` | Displays signal spectrum |
| `variable` | Stores fixed flowgraph parameters |
| `variable_qtgui_range` | Creates interactive GUI controls |

---

# 16. Important Parameter Relationships

## Sample Rate

The RF processing rate is:

```text
240 kS/s
```

while the audio processing rate is:

```text
48 kS/s
```

The NBFM blocks perform the necessary rate relationship between these domains.

---

## NBFM Parameters

Both the transmitter and receiver use:

```text
Maximum deviation = 5 kHz
Tau               = 75 µs
```

Maintaining corresponding NBFM parameters on the transmit and receive sides is important for consistent demodulation.

---

# 17. GUI Controls

The experiments expose several parameters as runtime controls.

### Receiver Gain

Used in the receiver flowgraphs.

```text
0–40
```

### Signal Gain

Used by the signal USRP transmit path.

```text
JammerTX   : 0–50
JammerRXTX : 0–40
```

### Noise Gain

Used by the noise USRP path.

```text
0–1
```

### Jam Power

Controls the amplitude of the generated Gaussian noise.

```text
0–10
```

### Cutoff Frequency

Controls the receive filter bandwidth.

### Transition Width

Controls the filter transition region.

---

# 18. Results and Observations

The GRC files establish several observable outputs.

## 18.1 Original NBFM Spectrum

The NBFM transmitter output is connected directly to a QT GUI Frequency Sink. Therefore, when the transmitter is running, the experiment provides a spectral representation of the generated FM signal.

---

## 18.2 Noise Spectrum

The Gaussian noise source is connected to a dedicated frequency sink. Changing its amplitude changes the level of the generated noise stream, allowing the spectral behavior to be observed.

---

## 18.3 Combined Signal + Noise

In `JammerRXTX.grc`, the NBFM signal and Gaussian noise are explicitly added using `blocks_add_xx`.

Therefore the flowgraph provides a direct observation point for the combined baseband signal.

---

## 18.4 Received Signal

The USRP Source provides the RF receive stream. The signal is filtered before being passed to the NBFM receiver.

The receive chain therefore allows the effect of the RF environment and configured signal/noise conditions to be observed at the receiver.

---

## 18.5 Demodulated Audio

The NBFM receiver converts the received complex signal back into audio at:

```text
48 kS/s
```

The recovered audio is sent to the system audio device.

---

## 18.6 What Cannot Be Determined from the GRC Files Alone

The flowgraphs do not contain recorded measurement results such as:

- BER
- SNR
- SINR
- RSSI measurement logs
- Audio quality metrics
- Communication range
- Packet loss
- Measured RF output power
- Measured interference range

Therefore these should not be reported as numerical experimental results unless they were measured separately during the physical experiment.

---

# 19. Comparison of the Three Flowgraphs

| Feature | `jammerRX.grc` | `JammerTX.grc` | `JammerRXTX.grc` |
|---|---|---|---|
| WAV Source | No | Yes | Yes |
| NBFM TX | No | Yes | Yes |
| Gaussian Noise | No | Yes | Yes |
| Signal + Noise Adder | No | No | Yes |
| USRP Source | Yes | Yes | Yes |
| USRP Sink | No | Yes | Yes |
| NBFM RX | Yes | Yes | Yes |
| Audio Output | Yes | Yes | Yes |
| Spectrum Displays | 1 | 3 | 4 |
| Main Purpose | Receive | Transmit/noise experiment | Combined experiment |

---

# 20. File Dependencies

The WAV source paths embedded in the supplied files are:

### `JammerTX.grc`

```text
D:\God Mode Begins - Masstamilan.MY.wav
```

### `JammerRXTX.grc`

```text
I:\SKILLS\SDDR RADIO\PROJECT\bassbasic-new-edm-music-beet-mr-sandeep-rock-141616 (1).wav
```

These are absolute paths from the original development machine. When the flowgraphs are moved to another computer, the paths must be updated to point to valid WAV files.

---

# 21. Hardware Configuration Notes

The GRC files contain specific USRP serial identifiers.

For example, the receiver flowgraph contains a specific device identifier for its USRP Source, while the combined and transmitter flowgraphs contain additional device identifiers.

These identifiers are not universal.

When moving the project to different hardware, the UHD device configuration should be checked using the installed UHD tools and the corresponding USRP should be selected in GNU Radio.

---

# 22. Recommended Laboratory Setup

For safe experimentation, use a controlled RF environment.

A preferred laboratory configuration is:

```text
USRP TX
   │
   ▼
Attenuation / Controlled RF Path
   │
   ▼
USRP RX
```

The experiment should be conducted using appropriate attenuation, shielding, dummy loads, or other authorized laboratory arrangements rather than transmitting uncontrolled interference into occupied spectrum.

---

# 23. Learning Outcomes

This project demonstrates practical implementation of:

1. Audio-to-IQ conversion using NBFM.
2. IQ-to-audio demodulation.
3. Complex Gaussian noise generation.
4. Baseband signal addition.
5. FIR filtering.
6. USRP hardware integration through UHD.
7. Real-time spectrum visualization.
8. Runtime parameter control using QT GUI ranges.
9. Separation of transmit, receive, and combined processing chains.
10. Observation of signal and noise behavior at multiple points in a communication system.

---

# 24. Conclusion

The three GNU Radio flowgraphs provide a progressive implementation of an NBFM-based RF experiment.

`jammerRX.grc` focuses on the **receiver and demodulation chain**.

`JammerTX.grc` focuses on **NBFM signal generation, configurable Gaussian-noise generation, and separate USRP transmit paths**.

`JammerRXTX.grc` combines the major elements into a **single experimental flowgraph**, including signal generation, noise generation, signal/noise combination, RF transmission, RF reception, filtering, demodulation, and spectrum visualization.

Together, the flowgraphs demonstrate how GNU Radio can be used to construct and inspect a complete SDR signal-processing chain from **audio → NBFM → RF/USRP → filtering → NBFM demodulation → audio**, while also providing controlled signal/noise experimentation.

> **Safety reminder:** RF interference experiments should remain within controlled, authorized laboratory conditions and applicable spectrum regulations.
