# CANFLY Cosmic Node

**Open experimental platform for detecting and studying cosmic-ray particles**

> First experiment: ground-based cosmic muon detector
> Status: **PLANNING / PROCUREMENT**
> Target hardware readiness: **June 2027**

---

## 1. Project idea

**CANFLY Cosmic Node** is an open experimental project for building a small, reproducible scientific instrument capable of detecting particles originating from cosmic-ray interactions.

The first experiment is intentionally simple:

> **Detect cosmic-ray muons at ground level, record individual events, characterize detector noise, and build a scientifically usable dataset.**

The project does **not** claim to detect dark matter, dark energy, or particles originating directly from a particular distant astronomical object.

The first objective is much more fundamental:

> **Learn to build, calibrate, operate and analyze a real particle detector.**

Once this capability is established, the same measurement architecture can be expanded into a directional muon telescope, a multi-detector cosmic-ray network, and eventually a spaceborne instrument.

---

# 2. Why muons?

Primary cosmic rays continuously interact with nuclei in Earth's atmosphere.

These interactions produce secondary particle cascades containing pions and kaons, whose decays produce muons.

Many of these muons have sufficient energy to reach the Earth's surface.

Muon detection therefore provides a direct and practical way to build an experimental connection between a laboratory on Earth and high-energy processes occurring in the atmosphere and beyond.

A typical ground-level cosmic muon has an energy of several GeV. CAEN's educational documentation describes an average energy of approximately 4 GeV at sea level and places typical muon production high in the atmosphere, around 15 km.

Important distinction:

**The detector does not tell us that a particular muon came from a particular star or galaxy.**

The first experiment measures the local flux of secondary cosmic-ray muons.

Sources:

* CAEN Educational — Muons Detection
* CAEN Educational — Muons Vertical Flux
* Particle Data Group references cited by CAEN

---

# 3. Scientific question

The initial scientific question is deliberately modest:

> **Can a compact, low-cost detector based on plastic scintillators and SiPMs continuously identify cosmic-ray muon candidates while suppressing detector noise and random coincidences?**

After this has been demonstrated, additional questions become possible:

* What is the measured muon count rate?
* How stable is the rate over hours, days and months?
* How does the rate depend on detector geometry?
* How does it depend on atmospheric pressure?
* How does it depend on altitude?
* How does it depend on detector orientation?
* What is the detector's efficiency?
* What is the accidental coincidence rate?
* Can particle trajectories be reconstructed?
* Can multiple geographically separated detectors identify correlated events?
* Can the system eventually be adapted for spaceflight?

---

# 4. What this project is NOT

This project is **not initially**:

* a dark-matter detector;
* a dark-energy detector;
* a gamma-ray observatory;
* a neutrino detector;
* an astronomical telescope;
* a source-identification system for individual cosmic rays;
* a replacement for professional cosmic-ray observatories.

The project should not make claims stronger than the measurements support.

The first successful result is simply:

> **We detected cosmic-ray particle events with a reproducible instrument and quantified the detector response.**

That is already a real experimental result.

---

# 5. Experimental principle

The first detector consists of two independent scintillation detector modules.

Each module contains:

```text
Cosmic particle
      ↓
Plastic scintillator
      ↓
Scintillation photons
      ↓
SiPM
      ↓
Preamplifier / readout
      ↓
Discriminator
      ↓
Digital pulse
```

The two modules are positioned one above the other.

A particle passing through both detectors produces approximately:

```text
TOP       ─────────●
                   │
                   │ particle
                   │
BOTTOM    ─────────●
```

The electronics searches for two detector pulses occurring within a predefined coincidence window.

The event is accepted when:

```text
TOP = 1
BOTTOM = 1
Δt < coincidence_window
```

This is important because individual SiPMs generate spontaneous dark-count pulses and other electronic noise.

A coincidence between two independent detectors strongly suppresses random single-detector events.

CAEN explicitly uses double scintillator coincidence for cosmic-muon detection and describes its role in reducing spurious events and selecting the detector's effective solid angle.

---

# 6. Detector architecture

Initial architecture:

```text
                    COSMIC PARTICLE
                          ↓
                 ┌─────────────────┐
                 │ SCINTILLATOR #1 │
                 │      + SiPM     │
                 └────────┬────────┘
                          │
                     analog pulse
                          │
                    ┌─────▼─────┐
                    │ READOUT #1│
                    └─────┬─────┘
                          │
                    DISCRIMINATOR
                          │
                          │
                     ┌────▼────┐
                     │         │
                     │  DAQ    │
                     │         │
                     │ STM32   │
                     │         │
                     └────┬────┘
                          │
                     USB / UART
                          │
                     Raspberry Pi
                          │
                    DATA STORAGE
                          │
             ┌────────────┴────────────┐
             │                         │
          RAW DATA                 ANALYSIS
```

The second detector has the same architecture:

```text
SCINTILLATOR #2
      ↓
     SiPM
      ↓
   READOUT #2
      ↓
 DISCRIMINATOR
      ↓
    STM32
```

The final trigger is based on coincidence between the two detector channels.

---

# 7. Initial hardware

## 7.1 Scintillators

Quantity:

**2**

Type:

**plastic scintillator**

Candidate material:

* EJ-200
* EJ-204
* equivalent plastic scintillator

Initial target dimensions:

approximately:

**50 × 50 × 10 mm**

Exact dimensions may be changed after component selection.

The two scintillators should preferably be identical.

---

## 7.2 Silicon Photomultipliers

Quantity:

**2**

Type:

**SiPM**

Target:

approximately 6 × 6 mm active area.

The initial implementation should preferably use commercially available SiPM modules or readout boards rather than designing the complete SiPM bias/readout circuit from scratch.

The purpose of the first prototype is to establish the physical experiment before optimizing the electronics.

---

## 7.3 SiPM readout / preamplifier

Quantity:

**2**

Required functions:

* SiPM bias
* signal extraction
* amplification
* suitable output for oscilloscope/discriminator

The exact board will be selected according to the chosen SiPM.

Compatibility between:

```text
SiPM
+
bias voltage
+
preamplifier
+
connector
+
oscilloscope/discriminator
```

must be verified before purchase.

---

## 7.4 Discriminator

Required:

**2-channel discriminator**

Function:

Convert analog detector pulses into digital events.

The threshold must be adjustable.

The threshold is an experimental parameter and will be calibrated rather than arbitrarily selected.

The detector must characterize:

```text
threshold
        ↓
single-channel count rate
        ↓
coincidence count rate
        ↓
random coincidence rate
```

CAEN specifically demonstrates measuring dark-count rate as a function of discriminator threshold and adjusting the threshold to suppress random coincidences.

---

## 7.5 DAQ microcontroller

Initial choice:

**STM32**

The STM32 is responsible for:

* detecting digital pulses;
* measuring event timing;
* counting single-channel events;
* identifying coincidences;
* generating event records;
* communicating with the host computer.

A 2026 CERN conference contribution describes a compact SiPM cosmic-ray system using an STM32 microcontroller and hardware timers for real-time event counting and coincidence processing, making STM32 a reasonable architecture for this class of instrument.

---

## 7.6 Raspberry Pi

Quantity:

**1**

Purpose:

The Raspberry Pi is the scientific data logger rather than the fast detector.

Responsibilities:

* receive events from STM32;
* maintain long-term storage;
* maintain local database;
* collect environmental measurements;
* synchronize files;
* generate monitoring plots;
* perform preliminary analysis;
* create backups.

---

## 7.7 GNSS receiver

Quantity:

**1**

Initial purpose:

* UTC time reference;
* geographic position;
* optional 1-PPS timing signal.

A normal GNSS receiver is sufficient for the first experiment.

A precision timing architecture can be introduced later.

---

## 7.8 Environmental sensors

Initial sensor:

**BME280 or equivalent**

Measurements:

* temperature;
* atmospheric pressure;
* humidity.

Atmospheric pressure is particularly important because the amount of atmosphere above the detector affects the secondary cosmic-ray population reaching the ground.

Environmental parameters therefore belong in the scientific dataset rather than being treated as irrelevant housekeeping.

---

## 7.9 Magnetometer

Quantity:

**1**

Possible devices:

* BMM150
* LIS3MDL
* equivalent 3-axis magnetometer

Purpose:

Record the local magnetic environment.

It is not initially used as a dark-matter detector.

It is an auxiliary variable for later correlation analysis and environmental characterization.

---

## 7.10 Laboratory power supply

Required:

adjustable laboratory DC power supply with current limiting.

If an appropriate laboratory power supply is already available, a new unit is unnecessary.

Stable detector bias is important for reproducible SiPM operation.

---

## 7.11 Oscilloscope

Recommended minimum:

**100 MHz bandwidth**

Preferably:

**≥1 GS/s**

Purpose:

The oscilloscope is primarily a development and calibration instrument.

It allows direct observation of:

* SiPM pulses;
* amplifier output;
* baseline noise;
* pulse amplitude;
* pulse width;
* discriminator threshold;
* electrical interference;
* coincidence timing.

The oscilloscope is not expected to be part of the final autonomous detector.

---

## 7.12 Optical coupling material

Required:

**optical coupling grease**

Purpose:

Improve optical coupling between the scintillator and SiPM.

The optical interface must remain mechanically stable and reproducible.

---

## 7.13 RF/electrical connections

Required as appropriate to the selected electronics:

* BNC cables;
* SMA cables;
* MCX adapters/cables;
* 50 Ω terminators;
* suitable coaxial connectors.

Exact connector requirements will be finalized after the SiPM/readout and discriminator models are selected.

---

## 7.14 Light-tight enclosure

The detector must be protected from ambient light.

Possible construction:

```text
scintillator
     +
SiPM
     ↓
light-tight wrapping
     ↓
mechanical enclosure
```

The enclosure must allow:

* optical isolation;
* mechanical stability;
* access for calibration;
* cable routing;
* reproducible detector geometry.

---

## 7.15 Mechanical frame

The first detector should have adjustable spacing.

Example:

```text
       ┌───────────────┐
       │ SCINTILLATOR 1│
       └───────────────┘
              │
              │ adjustable
              │ distance
              │
       ┌───────────────┐
       │ SCINTILLATOR 2│
       └───────────────┘
```

A 3D-printed frame is acceptable.

The geometry must be documented precisely.

---

# 8. Minimum hardware BOM

Initial procurement list:

| Component                   | Quantity | Required    |
| --------------------------- | -------: | ----------- |
| Plastic scintillator        |        2 | YES         |
| SiPM                        |        2 | YES         |
| SiPM readout/preamplifier   |        2 | YES         |
| 2-channel discriminator     |        1 | YES         |
| STM32 development board     |        1 | YES         |
| Raspberry Pi                |        1 | YES         |
| GNSS receiver               |        1 | RECOMMENDED |
| BME280/environment sensor   |        1 | RECOMMENDED |
| 3-axis magnetometer         |        1 | RECOMMENDED |
| Laboratory power supply     |        1 | YES*        |
| Oscilloscope                |        1 | YES*        |
| Optical coupling grease     |        1 | YES         |
| Coaxial cables/adapters     |      set | YES         |
| 50 Ω terminators            |      2–3 | RECOMMENDED |
| Light-tight material        |      set | YES         |
| 3D-printed mechanical frame |        1 | YES         |

`*` Only if suitable equipment is not already available.

---

# 9. Expected first result

The first successful measurement should produce something like:

```text
TIME                  TOP       BOTTOM    COINCIDENCE

2027-06-xx 12:00:01   124       119       8
2027-06-xx 12:01:01   130       125       9
2027-06-xx 12:02:01   127       121       7
...
```

The exact rates are not assumed in advance.

The detector must measure them.

A scientifically useful result is not simply:

> "The detector counted particles."

It is:

> "The detector measured a stable coincidence population that is distinguishable from independently measured single-channel noise and accidental coincidence background."

---

# 10. Experimental stages

## Stage 0 — Documentation

Before hardware assembly:

* freeze the initial detector design;
* record every component;
* record manufacturer and part number;
* record SiPM characteristics;
* record scintillator dimensions;
* record wiring;
* record firmware version;
* create the data schema;
* create calibration procedures.

No scientific measurement should exist without a record of the instrument configuration that produced it.

---

## Stage 1 — SiPM characterization

Before attaching the scintillator:

```text
SiPM
 ↓
readout
 ↓
oscilloscope
```

Measure:

* baseline;
* dark-count rate;
* pulse amplitude distribution;
* response to threshold changes;
* temperature dependence.

This establishes the detector's electronic baseline.

---

## Stage 2 — Single detector

Build one complete detector.

Measure:

```text
single-channel count rate
vs.
discriminator threshold
```

The goal is to understand the detector before attempting coincidence measurements.

---

## Stage 3 — Two detectors

Build the second detector.

Measure independently:

```text
R1 = detector 1 rate

R2 = detector 2 rate
```

Then activate coincidence:

```text
R12 = coincidence rate
```

---

## Stage 4 — Random coincidence characterization

This is mandatory.

Two detectors can generate accidental coincidences even when no particle traverses both.

The experiment must therefore estimate:

```text
R_random
```

and compare it with:

```text
R_cosmic_candidate
```

CAEN has a dedicated experiment specifically for random coincidence measurements because this effect is fundamental to coincidence detectors.

---

## Stage 5 — Cosmic muon detection

Place the two detectors vertically aligned.

Record:

```text
single rate #1
single rate #2
coincidence rate
temperature
pressure
humidity
timestamp
detector configuration
```

Run continuously.

Initial target:

**24-hour continuous dataset**

Then:

**7 days**

Then:

**30 days**

---

# 11. Geometry experiments

After basic detection works, change the geometry systematically.

### Experiment A — Maximum overlap

```text
┌───────────────┐
│       #1      │
└───────────────┘
┌───────────────┐
│       #2      │
└───────────────┘
```

Purpose:

maximize acceptance.

---

### Experiment B — Increased separation

```text
┌───────────────┐
│       #1      │
└───────────────┘

       ↑

     distance

       ↓

┌───────────────┐
│       #2      │
└───────────────┘
```

Purpose:

reduce the accepted solid angle and make the telescope more directional.

CAEN explicitly describes changing detector separation as a way to modify the solid angle and directional selection.

---

### Experiment C — Rotation

Rotate the entire detector.

Measure:

```text
count rate
vs.
zenith angle
```

This begins transforming the system from a simple particle counter into a directional cosmic-ray instrument.

---

# 12. Environmental correlations

Every long measurement should include:

```text
timestamp
temperature
pressure
humidity
magnetic field
detector voltage
threshold
detector geometry
```

Later analysis can test:

```text
muon rate
    vs.
atmospheric pressure
```

and:

```text
muon rate
    vs.
temperature
```

and:

```text
muon rate
    vs.
orientation
```

The purpose is not to assume a correlation exists.

The purpose is to **measure whether it exists**.

---

# 13. Data format

Every detected event should receive a unique identifier.

Example:

```json
{
  "event_id": 184729,
  "timestamp_utc": "2027-06-18T12:43:17.382941Z",
  "detector_top": true,
  "detector_bottom": true,
  "delta_t_ns": 37,
  "top_amplitude": 0.412,
  "bottom_amplitude": 0.387,
  "temperature_c": 22.4,
  "pressure_hpa": 1008.2,
  "humidity_percent": 48.1,
  "magnetic_field_ut": 48.7,
  "threshold_top": 31,
  "threshold_bottom": 31,
  "firmware_version": "0.1.0",
  "configuration_id": "MUON-V1-001"
}
```

The exact schema will evolve.

Raw data should never be destroyed after calibration.

---

# 14. Data layers

The repository should distinguish:

```text
raw/
calibrated/
derived/
analysis/
plots/
metadata/
```

### RAW

Data exactly as produced by the detector.

### CALIBRATED

Data after applying documented calibration constants.

### DERIVED

Calculated quantities such as:

* rates;
* coincidence rates;
* efficiencies;
* pressure-corrected quantities;
* angular distributions.

### ANALYSIS

Reproducible analysis code.

### PLOTS

Generated figures.

### METADATA

Instrument configuration and calibration information.

---

# 15. Repository structure

Recommended structure:

```text
canfly-cosmic-node/
│
├── README.md
│
├── docs/
│   ├── physics.md
│   ├── detector.md
│   ├── electronics.md
│   ├── calibration.md
│   ├── experiments.md
│   └── data-format.md
│
├── hardware/
│   ├── bom/
│   ├── schematics/
│   ├── pcb/
│   └── mechanical/
│
├── firmware/
│   └── stm32/
│
├── software/
│   ├── collector/
│   ├── database/
│   └── monitoring/
│
├── analysis/
│   ├── calibration/
│   ├── coincidence/
│   ├── flux/
│   └── environmental/
│
├── data/
│   └── README.md
│
├── notebooks/
│
├── experiments/
│   ├── 001-single-detector/
│   ├── 002-two-detector/
│   ├── 003-coincidence/
│   ├── 004-muon-rate/
│   ├── 005-pressure/
│   └── 006-angular-response/
│
└── LICENSE
```

Large datasets should not necessarily be stored directly in Git.

---

# 16. Experimental record

Every experiment should have a unique identifier.

Example:

```text
EXP-001
```

with:

```text
date
location
detector configuration
scintillator dimensions
SiPM model
bias voltage
threshold
coincidence window
detector spacing
orientation
environment
firmware version
software version
operator
duration
raw-data location
```

This makes the experiment reproducible.

---

# 17. Scientific integrity rules

The project follows several rules.

### Rule 1

Never call an unexplained event "new physics".

First eliminate:

* electronic noise;
* SiPM dark counts;
* accidental coincidences;
* power-supply noise;
* electromagnetic interference;
* temperature effects;
* timing errors;
* software errors;
* mechanical changes;
* environmental correlations.

### Rule 2

Never modify raw data.

Calibration creates a new dataset.

### Rule 3

Every change to the detector configuration receives a new configuration ID.

### Rule 4

Every firmware change receives a version.

### Rule 5

Every analysis result must be reproducible from stored data.

### Rule 6

Unexpected results must trigger additional measurements rather than immediate interpretation.

---

# 18. Success criteria for v1

Version 1 is considered successful when the system can:

* continuously operate for at least 24 hours;
* record individual detector events;
* measure independent detector rates;
* measure coincidence events;
* quantify accidental coincidence background;
* demonstrate a statistically significant excess of physical coincidences over the measured accidental background;
* reproduce the result on another measurement run;
* store timestamped environmental data;
* preserve raw data;
* regenerate analysis plots from the stored dataset.

The project does **not** require discovering anything unknown to be considered successful.

---

# 19. Future development

Once v1 is stable, the project can evolve.

```text
                 CANFLY COSMIC NODE
                         │
                         ▼
                ┌─────────────────┐
                │  MUON DETECTOR  │
                │       V1        │
                └────────┬────────┘
                         │
                         ▼
                ┌─────────────────┐
                │ MUON TELESCOPE  │
                │       V2        │
                └────────┬────────┘
                         │
                         ▼
                ┌─────────────────┐
                │ MULTI-DETECTOR  │
                │     NETWORK     │
                └────────┬────────┘
                         │
                         ▼
                ┌─────────────────┐
                │  SPACE-QUALIFIED│
                │     CONCEPT     │
                └─────────────────┘
```

Potential future instruments include:

* three-layer muon telescope;
* four-layer particle tracker;
* cosmic shower detector;
* geographically distributed detector network;
* high-altitude detector;
* balloon experiment;
* CubeSat experiment.

---

# 20. First scientific roadmap

### 2026

Project definition.

Physics study.

Hardware selection.

Procurement.

Electronics architecture.

Software architecture.

Data format.

Simulation and analysis development.

---

### June 2027

Target:

**first complete ground detector**

Initial tasks:

```text
power-on
↓
SiPM characterization
↓
single detector
↓
second detector
↓
coincidence
↓
first cosmic events
```

---

### 2027 — Phase 1

Stable 24-hour operation.

---

### 2027 — Phase 2

7-day continuous dataset.

---

### 2027 — Phase 3

30-day dataset.

---

### 2027+ — Phase 4

Directional measurements.

Environmental correlation.

Detector efficiency.

Long-term stability.

---

# 21. Reference experiments

The project deliberately follows established detector principles rather than inventing an untested detection method.

### CAEN — Muons Detection

CAEN demonstrates cosmic-muon detection using a plastic scintillating tile directly coupled to a Silicon Photomultiplier.

The documentation also describes SiPM dark-count characterization and discriminator-threshold optimization.

### CAEN — Muons Vertical Flux

CAEN demonstrates measuring the vertical cosmic-muon flux and comparing measured rates with expected flux.

### CAEN — Random Coincidence

CAEN provides a dedicated experiment for quantifying accidental coincidences between detector channels.

### CAEN — Detection Efficiency

CAEN demonstrates efficiency measurements using multiple scintillator coincidence configurations.

### CAEN — Cosmic Shower Detection

CAEN demonstrates detection of cosmic-ray showers using multiple scintillator coincidences.

### CERN / TIFR — Compact SiPM cosmic-ray systems

Recent detector-development work demonstrates compact plastic-scintillator/SiPM systems with discriminator electronics and STM32-based data acquisition.

### CosmicWatch

The CosmicWatch project demonstrates a compact educational cosmic-ray detector based on a small plastic scintillator, SiPM and microcontroller.

---

# 22. Project philosophy

The project begins with a deliberately small question:

> **Can we build a reliable instrument that sees particles we cannot see?**

Then:

> **Can we measure them accurately?**

Then:

> **Can we understand the detector well enough to trust the data?**

Then:

> **Can multiple instruments see the same physical phenomenon?**

Only after those questions have been answered should the project attempt more ambitious measurements.

The long-term goal is not to build a device that produces mysterious numbers.

The goal is to build a device whose numbers can be **trusted, reproduced and independently analyzed**.

---

# 23. Project name

Recommended repository:

```text
canfly-cosmic-node
```

Possible alternatives:

```text
canfly-cosmic
canfly-muon-node
canfly-cosmic-detector
canfly-cosmic-lab
cosmic-node
```

Recommended:

**`canfly-cosmic-node`**

because the project is intended to grow beyond the first muon detector.

---

# 24. Current status

```text
[██████████] Project concept       DONE
[██████████] Physics definition    DONE
[██████████] Initial architecture DONE
[██████████] Initial BOM            DONE
[░░░░░░░░░░] Component selection    NEXT
[░░░░░░░░░░] Procurement            PLANNED
[░░░░░░░░░░] Electronics            PLANNED
[░░░░░░░░░░] Firmware               PLANNED
[░░░░░░░░░░] Detector assembly      PLANNED
[░░░░░░░░░░] Calibration            PLANNED
[░░░░░░░░░░] First cosmic event     PLANNED
[░░░░░░░░░░] Long-term dataset     PLANNED
```

**Target first complete ground experiment: June 2027.**

---

## Scientific references

The initial project design is based on established cosmic-ray and detector methods described by:

* CAEN Educational — Muons Detection
* CAEN Educational — Muons Vertical Flux on Horizontal Detector
* CAEN Educational — Random Coincidence
* CAEN Educational — Detection Efficiency
* CAEN Educational — Cosmic Shower Detection
* CERN / TIFR detector-development work on compact plastic-scintillator + SiPM systems
* CosmicWatch educational cosmic-ray detector project

All quantitative claims and detector parameters should be checked against the primary documentation before being frozen into the hardware design.
