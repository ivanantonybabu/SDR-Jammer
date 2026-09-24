# GNU Radio DFR / RF Interference Experiments

This project contains three GNU Radio Companion (GRC) flowgraphs developed to study an FM signal, controlled Gaussian-noise injection, RF reception, spectrum observation, and USRP-based transmit/receive paths. The three flowgraphs are `JammerTX.grc`, `jammerRX.grc`, and `JammerRXTX.grc`. Together, they represent three stages of the experiment: a transmit-side flowgraph that generates an NBFM signal and a configurable noise signal, a receive-side flowgraph that captures and demodulates an NBFM signal, and a combined transmit/receive flowgraph that places the signal and noise paths in the same experiment while providing separate spectrum displays. The designs use UHD USRP Source/Sink blocks, NBFM modulation/demodulation, FIR low-pass filtering, a Gaussian complex noise source, WAV audio, audio sinks, and QT GUI frequency sinks.

> **Safety / regulatory note:** These flowgraphs involve RF transmission and noise generation. They should be used only in a controlled laboratory setup, such as a properly attenuated/cabled test arrangement or an authorized RF test environment. Do not use them to interfere with licensed or public radio communications.

## Flowgraphs

### `JammerTX.grc`
Transmit-side experiment combining a WAV audio source, NBFM modulation, a configurable Gaussian complex-noise source, and two USRP transmit paths. The flowgraph also monitors the original NBFM signal and the generated noise in the frequency domain and provides local audio monitoring of the source and recovered receive path.

### `jammerRX.grc`
Receive-side experiment using a UHD USRP Source followed by a configurable FIR low-pass filter and NBFM demodulator. The recovered audio is sent to an audio sink, while a QT GUI frequency sink provides spectrum observation of the demodulated output.

### `JammerRXTX.grc`
Combined experiment containing both an NBFM transmit chain and an independent USRP receive chain. The WAV source is FM-modulated and combined with a configurable complex Gaussian-noise source. Separate USRP sinks are used for the signal and noise paths, while the receive side filters and demodulates the captured signal. Multiple frequency sinks show the original FM signal, noise, combined signal, and demodulated output.

## Main Concepts

- Narrowband FM (NBFM)
- WAV/audio signal processing
- Gaussian complex noise generation
- RF signal reception
- RF spectrum observation
- FIR low-pass filtering
- NBFM demodulation
- USRP / UHD integration
- QT GUI frequency-domain visualization
- Controlled RF interference experimentation

For a complete block-by-block explanation, signal flow, parameters, implementation details, and result interpretation, see **[DOCUMENTATION.md](./DOCUMENTATION.md)**.
