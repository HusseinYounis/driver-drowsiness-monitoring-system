# 😴 Driver Drowsiness Monitoring System

> **Senior Graduation Project** — Computer System Engineering

---

## 📋 Abstract

**Driver Drowsiness Monitoring System** is an intelligent, real-time safety system that detects early signs of driver drowsiness using **Electrooculography (EOG)** signals and **Machine Learning** techniques.

EOG electrodes placed around the driver's eyes acquire electrical signals associated with eye movements and eyelid activity. A **BioAmp EXG Pill** module amplifies and conditions these biological signals, while an **ESP32 microcontroller** samples and transmits the data to a computer for real-time processing. Python-based signal processing extracts relevant features — blinking frequency, signal amplitude, eye closure duration — and feeds them to a trained ML classifier. When drowsiness patterns are detected, the system immediately triggers an alert via a **buzzer or vibration motor** to warn the driver.

The result is an affordable, practical prototype that unifies biomedical signal acquisition, embedded systems, digital signal processing, and artificial intelligence into a single driver monitoring solution.

---

## 🎯 Problem Statement

Driver drowsiness is a serious and progressively worsening road safety problem. As fatigue builds, a driver's attention and reaction time degrade — often without the driver recognising the danger. This project addresses four core issues:

1. **Gradual, unnoticed fatigue** — Drowsiness develops slowly; drivers rarely self-diagnose in time to act.
2. **Camera-system limitations** — Existing camera-based approaches are sensitive to lighting, sunglasses, and face obstruction.
3. **No physiological grounding** — Visual-only systems infer state from appearance, not from measurable eye-activity signals.
4. **High cost and complexity** — Commercial driver monitoring systems are expensive and not accessible for everyday vehicles.

---

## 🔍 How This System Differs from Related Work

Most existing driver monitoring solutions rely on cameras and computer vision. This project takes a fundamentally different approach:

### 1. Physiological Signal vs. Visual Inspection
Instead of analysing a camera feed of the driver's face, the system reads **electrical signals produced by eye movements** (EOG). This makes it immune to lighting conditions, sunglasses, and partial face obstruction.

### 2. Real Biometric Measurement
EOG captures the electrical potential difference between the cornea and retina, providing a direct, hardware-level measurement of eye activity — not an inferred estimate from pixels.

### 3. Dual Alert Mechanism
The system activates both an **audible buzzer** and a **vibration motor**, providing redundant alerts that work even when the driver is wearing hearing protection or has hearing impairment.

### 4. Low-Cost, Open Hardware
Built around the **BioAmp EXG Pill** and an **ESP32**, the total hardware cost is under $135 — a fraction of commercial systems.

### Summary Table

| Aspect | Typical Camera-Based Approaches | This System |
|--------|---------------------------------|-------------|
| Sensing method | Camera + face/eye detection CV model | EOG electrodes — direct biometric signal |
| Affected by lighting? | Yes | No |
| Affected by sunglasses? | Yes | No |
| Signal type | Visual / pixel-level | Electrical / physiological |
| Alert mechanism | Audio only or dashboard light | Buzzer + vibration motor |
| Hardware cost | High (camera, Raspberry Pi, GPU) | Low (~$73–$135, ESP32-based) |
| ML input | Image frames | Time-series EOG features |

---

## 🏗️ System Architecture

```
[Biometric Sensing]      [Signal Conditioning]    [Microcontroller]     [Computer]          [Alert]
  EOG Electrodes    ──>  BioAmp EXG Pill      ──>  ESP32          ──>  Python App     ──>  Buzzer
  (around eyes)          (amplify + filter)        (ADC + Serial)      (DSP + ML)          Vibration Motor
                                                                         │                  LED Indicator
                                                                         └──> GUI Dashboard
```

The pipeline has four active stages:
- **Acquisition stage** — EOG electrodes capture raw eye-movement potentials
- **Conditioning stage** — BioAmp EXG Pill amplifies and band-pass filters the µV-level signal
- **Transmission stage** — ESP32 samples the conditioned analog signal via ADC and streams digital samples to the computer over serial
- **Processing & classification stage** — Python DSP pipeline filters, segments, extracts features, and classifies; triggers alert command back to ESP32

---

## ⚙️ Hardware Components

| Component | Function | Notes |
|-----------|----------|-------|
| **EOG Electrodes** | Acquire electrical signals associated with eye movements | Placed around the eyes (above, below, and beside each eye) |
| **BioAmp EXG Pill** | Amplify and condition weak biological signals | Provides instrumentation-amp gain and band-pass filtering |
| **ESP32 Development Board** | Sample the conditioned analog signal via ADC and transmit to computer | Serial or USB communication; also controls alert components |
| **Buzzer** | Audible drowsiness warning alert | Activated by ESP32 on command from the Python app |
| **Vibration Motor** | Tactile drowsiness warning alert | Redundant alert for environments where audio may be masked |
| **LED Indicators** | Display system status and active warnings | Visual feedback on alert level |
| **Power Supply** | Power all hardware components | USB or battery pack |
| **Computer / Laptop** | Run signal processing and Machine Learning algorithms | Not included in hardware cost |

### BioAmp EXG Pill — Biological Signal Acquisition Front-End

The BioAmp EXG Pill is designed specifically for biopotential signal acquisition. Its key roles are:
- **Instrumentation amplifier** — rejects common-mode noise while amplifying the differential EOG signal (typical EOG amplitude: 50–3500 µV)
- **Band-pass filtering** — passes the EOG frequency band (0.5–40 Hz) and attenuates mains interference (50/60 Hz)
- **Right-leg drive circuit** — reduces common-mode noise caused by power-line interference

### ESP32 Firmware Highlights (Arduino / C++)
- Reads the conditioned EOG signal from an ADC pin (12-bit, 0–3.3 V)
- Streams samples at a configurable rate (default: 500 Hz) over USB serial
- Receives alert commands from the Python app and drives buzzer + vibration motor
- LED pin indicates system armed / drowsiness detected state

---

## 🧠 Signal Processing Pipeline

All signal processing runs in Python and follows a staged pipeline:

### Stage 1 — Preprocessing
| Technique | Purpose |
|-----------|--------|
| Band-pass filter (0.5–10 Hz) | Remove DC drift and high-frequency muscle noise |
| Notch filter (50 Hz) | Eliminate mains power-line interference |
| Baseline correction | Remove electrode offset drift over time |
| Signal normalisation | Scale amplitude to a consistent range across subjects |
| Segmentation | Divide continuous stream into fixed-length time windows |

### Stage 2 — Feature Extraction
| Feature | Description |
|---------|------------|
| **Blinking frequency** | Number of blinks per unit time — increases with fatigue |
| **Signal amplitude** | Peak-to-peak voltage of eye-movement events |
| **Eye closure duration** | Duration of eye-closure events (long closure → drowsy) |
| **Signal variance** | Variability of the EOG signal over a window |
| **Time-domain statistics** | Mean, RMS, skewness, kurtosis per segment |
| **Blink interval** | Time between consecutive blinks |

### Stage 3 — Classification
The extracted feature vector is passed to a trained Machine Learning classifier:

```python
from sklearn.ensemble import RandomForestClassifier

# Example feature vector: [blink_freq, amplitude, closure_duration, variance, rms]
X_train = [
    [0.3, 150, 0.2, 200, 130],  # Normal activity
    [0.8, 90,  0.6, 800, 300],  # Drowsy activity
]
y_train = [0, 1]  # 0 = Normal, 1 = Drowsy

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)
```

Algorithms evaluated:
- **Random Forest** — primary candidate; robust to sensor noise, requires no large dataset
- **Support Vector Machine (SVM)** — strong with high-dimensional feature vectors
- **K-Nearest Neighbors (KNN)** — simple baseline
- **Decision Tree** — interpretable comparison model

All four are trained and compared on the collected EOG dataset. The best-performing model is deployed in the real-time pipeline.

---

## 🚨 Alert System

When the classifier detects drowsiness, the Python application sends a command over serial to the ESP32, which activates:

| Alert Component | Trigger Condition | Action |
|-----------------|-------------------|--------|
| **Buzzer** | Drowsiness class predicted | Emits repeated beeps at increasing frequency |
| **Vibration Motor** | Drowsiness class predicted | Produces tactile vibration at the steering wheel or seat |
| **Red LED** | Drowsiness class predicted | Solid red — persistent until driver is alert again |
| **Green LED** | Normal state | Solid green — system armed and operating normally |

---

## 💻 Software Requirements

| Technology | Purpose |
|------------|---------|
| **Python** | Main programming language for signal processing and ML |
| **NumPy** | Numerical operations and signal manipulation |
| **SciPy** | Digital filtering (band-pass, notch) and signal analysis |
| **Pandas** | Organising and managing collected EOG datasets |
| **Scikit-learn** | Training and evaluating Machine Learning models |
| **Matplotlib** | Visualising raw and processed EOG signals |
| **Arduino IDE** | Programming the ESP32 firmware |
| **Tkinter / PyQt** | Developing the graphical monitoring interface |
| **pySerial** | Serial communication between Python and ESP32 |

### Running the System
```bash
# Install Python dependencies
pip install numpy scipy pandas scikit-learn matplotlib pyserial

# Run the main application (Senior 2 addition)
python main.py
```

---

## 🖥️ Graphical User Interface

A real-time monitoring GUI displays:

| Panel | Content |
|-------|---------|
| **Live EOG Signal** | Scrolling time-series plot of the raw and filtered EOG waveform |
| **Classification Result** | Current state — "Normal" or "Drowsy" |
| **Alert Status** | Active warning indicator and elapsed alert time |
| **Feature History** | Rolling chart of blinking frequency and amplitude |
| **System Log** | Timestamped record of all drowsiness events and alerts |

---

## 🗺️ Project Roadmap

### Senior 1 — Research & Design Phase ✅
- [x] Study EOG technology and biological signal acquisition fundamentals
- [x] Define system architecture and component selection
- [x] Identify hardware and software requirements
- [x] Design signal processing pipeline and ML classification approach
- [x] Define feature set for drowsiness detection
- [x] Write project proposal and obtain approval

### Senior 2 — Implementation & Validation Phase 🔜
- [ ] Procure hardware (BioAmp EXG Pill, ESP32, electrodes, buzzer, vibration motor)
- [ ] Wire and test the biomedical signal acquisition circuit
- [ ] Program ESP32 firmware for ADC sampling and serial streaming
- [ ] Collect EOG dataset from participants under different eye-activity conditions
- [ ] Implement and tune signal preprocessing pipeline (filter, normalise, segment)
- [ ] Extract feature set and build labelled training dataset
- [ ] Train and compare ML models (RF, SVM, KNN, Decision Tree)
- [ ] Integrate trained model into real-time Python pipeline
- [ ] Implement and test alert system (buzzer, vibration motor, LED)
- [ ] Build graphical monitoring GUI
- [ ] Validate system performance: classification accuracy, response time, reliability

---

## 💰 Estimated Budget

| Component | Estimated Cost (USD) |
|-----------|---------------------|
| EOG Electrodes | $5–10 |
| BioAmp EXG Pill | $30–40 |
| ESP32 Development Board | $8–12 |
| Buzzer | $2–3 |
| Vibration Motor | $3–5 |
| LED Indicators + Wires | $5–10 |
| Power Supply & Accessories | $10–15 |
| Additional Components | $10–40 |
| **Estimated Total** | **$73–135** |

> Computer / laptop assumed to be available; not included.

---

## ⚠️ Limitations and Safety Considerations

- EOG signals can be affected by muscle artifact, electrode movement, and electrical interference.
- Electrode placement and skin contact quality influence signal fidelity.
- EOG patterns vary between individuals — the model requires per-subject or cross-subject calibration.
- ML performance depends on dataset size and diversity.
- EOG does not directly measure sleepiness; it measures eye-activity indicators correlated with fatigue.
- The prototype is an **experimental research system** — it is not a certified vehicle safety device and requires further testing before any real-world deployment.
- All experiments involving human participants use low-voltage, isolated, and biologically safe equipment (IEC 60601 principles applied).

---

## 📁 Project Structure

```
driver-drowsiness-monitoring-system/
├── README.md            # This file
```

> **Senior 2 additions:** ESP32 firmware (`firmware/`), Python DSP + ML pipeline (`src/`), dataset (`data/`), trained models (`models/`), and GUI application (`app/`) will be added as development progresses.

---

## 🌍 Context

Developed in **Palestine** as a graduation project targeting a real-world road safety problem — combining biomedical electronics, embedded systems, digital signal processing, and artificial intelligence in an affordable, open-hardware prototype accessible for everyday vehicles.

---

## 👥 Team

| Student ID | Name |
|------------|------|
| 201910731 | Waleed Samara |
| 202110415 | Ali Aljamal |

**Supervisor:** Mr. Hussein Younis  
**Department:** Computer System Engineering  
**Faculty:** Faculty of Engineering and Information Technology  
**Institution:** Arab American University — Palestine  
**Academic Year:** 2026 – 2027
