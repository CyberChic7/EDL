# EDL
# Dual MAX30001 PTT / ECG Biomedical Monitor

A real-time biomedical monitoring project built around the **STM32F411 Black Pill** and **two MAX30001 analog front-end chips**. The firmware streams ECG and BioZ data to a laptop, where a desktop visualizer computes pulse transit time (PTT), pulse wave velocity (PWV), ECG morphology metrics, and battery/power telemetry. A Flask web dashboard mirrors the same live data and waveforms, so that is can be accessed by any device on the internet. 

## Overview

This system measures and visualizes signals from two MAX30001 devices:

* **Chip 1**: ECG1 + BioZ1
* **Chip 2**: BioZ2

The firmware performs onboard streaming and event flagging, including:

* **R-peak detection** from ECG1
* **APW1 foot detection** using an ECG-triggered cubic-polynomial method
* **APW2 foot detection** using a self-triggered asymmetric morphological V-shape detector
* **INA219 telemetry** for bus voltage, current, and power

On the laptop, the visualizer:

* Receives packets over **UDP**
* Computes **PTT** from detected foot markers
* Estimates **PWV** from PTT and a fixed sensor distance
* Optionally predicts **blood pressure** using a saved model
* Exports the live state to a JSON file used by the web dashboard

The web dashboard then displays:

* Heart rate
* Estimated blood pressure
* PTT and PWV
* Battery telemetry
* ECG morphology metrics
* Live ECG / APW waveforms

## Repository contents

* `firmware' : STM32 Black Pill firmware for the dual MAX30001 setup
* `compute_display.py`: desktop visualizer and UDP listener
* `edl_webserver.py`: Flask web dashboard
* `vitals_state.json`: shared state file created at runtime


### SPI mapping used in firmware

* **SCK** → PA5
* **MISO** → PA6
* **MOSI** → PA7
* **CS1** → PA0
* **CS2** → PB0

## Software flow

1. The STM32 samples ECG and BioZ from both MAX30001 chips.
2. The firmware packs the samples into a 36-byte packet and sends them over serial/USB.
3. `compute_display.py` receives the packets, parses them, and updates the live visualizer.
4. The visualizer calculates PTT, PWV, heart rate, morphology metrics, and writes them to `vitals_state.json`.
5. `edl_webserver.py` reads that JSON file and serves the web dashboard.


## Features

* Dual-channel ECG + BioZ acquisition
* Onboard DSP on the STM32
* Real-time PTT and PWV estimation
* Blood Pressure prediction from a trained model
* Live desktop plots and web dashboard
* Battery telemetry monitoring
* ECG morphology and HRV analysis

## Installation

### Python dependencies

Install the required Python packages on your laptop:

```bash
pip install numpy scipy pandas pyqtgraph pyqt6 flask pyngrok joblib scikit-learn
```

If you use a trained blood-pressure model, place it in the project folder as:

```bash
bp_xgboost_model.pkl
```

### Arduino / STM32 firmware

The firmware uses:

* `SPI.h`
* `Wire.h`
* `Adafruit_INA219`
* `protocentral_max30001`

Make sure those libraries are installed in your Arduino environment.

## How to run

### 1) Flash the STM32 firmware

Upload the Black Pill code to the STM32F411 board.

### 2) Start the laptop visualizer

Run:

```bash
compute_display.py
```

This opens the desktop GUI and listens for UDP packets.

### 3) Start the web server

In a second terminal, run:

```bash
python edl_webserver.py
```

This starts the Flask dashboard on port `5000`.

### 4) Open the dashboard

Use one of the printed URLs:

* Local: `http://localhost:5000`
* LAN IP: shown in the terminal
* Public ngrok URL: shown in the terminal if ngrok connects successfully. Use that link to access the dashboard from any device connected to the internet. 

## Runtime notes

* The visualizer and web dashboard must both be running for the full system to work.
* The web dashboard reads from `vitals_state.json`, so that file will be updated continuously by the desktop app.
* If the BP model file is missing, the system still runs; only the BP prediction card remains unavailable.
* The code is tuned for a sampling rate of **128 Hz**.

## Project purpose

This project was built as a real-time biomedical signal processing and visualization system for:

* ECG acquisition
* Bioimpedance-based pulse timing analysis
* PTT / PWV estimation
* Blood pressure estimation experiments
* Live monitoring through desktop and web interfaces

* Add a troubleshooting section for serial, UDP, and ngrok issues

## License

MIT License

---


