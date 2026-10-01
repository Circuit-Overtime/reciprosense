# ReciproSense
## Hardware-Aware Reciprocal Wi-Fi Sensing Using Two ESP8266 Nodes

### Research Goal
Investigate whether two low-cost ESP8266 radios can separate **environment-induced RF changes** from **hardware-induced RSSI drift** using:

- Bidirectional Wi-Fi measurements
- Multi-channel observations
- Temperature monitoring
- Voltage/current monitoring
- External timing reference
- Hardware-aware compensation

The primary research question is:

> Can two commodity ESP8266 nodes distinguish changes in the wireless propagation environment from variations caused by their own hardware?

---

# 1. Core Hypothesis

Conventional RSSI sensing often assumes:

```text
RSSI ≈ Environment
```

In reality:

```text
RSSI = Environment + Hardware + Noise
```

Possible hardware effects include:

- Temperature variation
- Supply-voltage variation
- Oscillator drift
- TX/RX asymmetry
- RF front-end drift
- Antenna/orientation effects

The proposed system models the environmental contribution as:

\[
EnvironmentalSignal \approx f(R_{AB}, R_{BA}, T_A, T_B, V_A, V_B, Clock_A, Clock_B)
\]

where:

- \(R_{AB}\): RSSI measured at node B when A transmits
- \(R_{BA}\): RSSI measured at node A when B transmits
- \(T_A, T_B\): node temperatures
- \(V_A, V_B\): node supply voltages
- \(Clock_A, Clock_B\): measured clock drift or timing offsets

---

# 2. Proposed Contributions

The project should target the following contributions:

1. A **bidirectional RSSI reciprocity model** for ESP8266-class radios.
2. Separation of:
   - Hardware-induced RF variation
   - Environment-induced RF variation
3. A **multi-channel reciprocal RF signature** without CSI.
4. Hardware-aware compensation using:
   - Temperature
   - Voltage/current
   - Clock drift
5. Controlled validation using material and liquid-state experiments.
6. An open dataset, firmware, and analysis pipeline.

---

# 3. System Architecture

```text
                    Laptop / PC
                 Experiment Controller
                        |
              Serial logging / commands
                        |
          +-------------+-------------+
          |                           |
          v                           v
      ESP8266 A                   ESP8266 B
      +--------+                  +--------+
      | Wi-Fi  |<--------------->| Wi-Fi  |
      | RSSI   |                  | RSSI   |
      | Timing |                  | Timing |
      +---+----+                  +---+----+
          |                           |
      Temp/V/I                    Temp/V/I
          |                           |
          +--------- RTC -------------+

                  RF test path
                       |
                       v
                 Test material
                       |
                    Servo
                 (optional)
```

---

# 4. Hardware

## Required
- 2 × ESP8266 boards
- 2 × temperature sensors
- 2 × INA219 or equivalent voltage/current sensors
- 1 × low-cost RTC/reference-clock module
- Breadboards
- Jumper wires
- Stable USB power supplies

## Recommended
- 1 × servo motor for repeatable material positioning
- Plastic/acrylic material holder
- Ruler/measuring tape
- Fixed mounts for the ESP8266 boards

---

# 5. Measurements to Collect

Every transmitted packet should generate a record containing as many of the following as practical:

```text
timestamp
experiment_id
trial_id
material
position_cm
fill_percentage
channel
tx_node
rx_node
packet_number
rssi
packet_received
packet_latency
temperature_tx
temperature_rx
voltage_tx
voltage_rx
current_tx
current_rx
clock_offset_tx
clock_offset_rx
```

Derived features:

\[
R_C = \frac{R_{AB}+R_{BA}}{2}
\]

\[
R_D = R_{AB}-R_{BA}
\]

Additional statistics:

- Mean RSSI
- Median RSSI
- RSSI variance
- RSSI standard deviation
- Packet Error Rate (PER)
- Packet latency
- Timing variance
- Clock drift

Packet Error Rate:

\[
PER = \frac{PacketsSent-PacketsReceived}{PacketsSent}
\]

---

# 6. Phase 0 — Hardware Validation

Place the two ESP8266 boards approximately 1 meter apart.

```text
ESP A -------------------- ESP B
              1 m
```

Keep the RF path empty.

Run the system for 30–60 minutes.

Record:

- RSSI A → B
- RSSI B → A
- Temperature A
- Temperature B
- Voltage
- Current
- Clock reference
- Packet loss
- Timing

Goal:

Establish the natural noise floor and determine whether:

\[
R_{AB} \neq R_{BA}
\]

even under static environmental conditions.

Example:

```text
A -> B RSSI = -48 ± 1.8 dBm
B -> A RSSI = -51 ± 2.1 dBm
```

---

# 7. Phase 1 — ESP8266 Hardware Characterization

The goal of this phase is to understand how much apparent RF variation is produced by the hardware itself.

## 7.1 Temperature Test

Keep the environment unchanged.

Slightly vary the temperature of ESP8266 A while keeping B stable.

Example temperatures:

```text
25 °C
30 °C
35 °C
40 °C
```

Do not exceed safe operating limits.

Measure:

\[
RSSI_{AB}(T_A)
\]

\[
RSSI_{BA}(T_A)
\]

Then repeat for ESP8266 B.

Estimate:

\[
\frac{\partial RSSI}{\partial T}
\]

Also measure clock drift versus temperature.

---

# 8. Phase 2 — Voltage Variation Test

Keep the RF environment unchanged.

Safely vary the input power within the board's supported range.

Measure:

\[
RSSI(V)
\]

\[
ClockDrift(V)
\]

Goal:

Determine whether changing supply conditions causes measurable RF or timing drift.

---

# 9. Phase 3 — Clock Characterization

Use the RTC/reference clock as an external timing reference.

Compare ESP8266 internal timing against the reference.

Example:

```text
ESP elapsed time       = 10.00032 s
Reference elapsed time = 10.00000 s
```

Clock drift:

\[
Drift =
\frac{t_{ESP}-t_{REF}}{t_{REF}}
\times 10^6
\]

For the example:

\[
Drift = 32\ ppm
\]

Correlate clock drift with:

- Temperature
- Voltage
- RSSI variation

---

# 10. Phase 4 — Basic Environmental Sensing

Begin with an easy and controlled target: water fill level.

## Dataset A

```text
Class 0 = Air / no object
Class 1 = Empty plastic bottle
Class 2 = 25% water
Class 3 = 50% water
Class 4 = 75% water
Class 5 = 100% water
```

Place the bottle midway between the two nodes.

```text
A ------------ [ bottle ] ------------ B
```

For every state:

- Sweep supported Wi-Fi channels
- Send 100–500 packets per channel
- Measure both directions
- Repeat at least 20 independent trials

---

# 11. Multi-Channel Measurement

For every channel \(f_i\), measure:

\[
R_{AB}(f_i)
\]

\[
R_{BA}(f_i)
\]

\[
R_C(f_i)
\]

\[
R_D(f_i)
\]

A multi-channel environmental response vector can then be represented as:

\[
X =
[
R_C(1),
R_C(2),
...,
R_C(n)
]
\]

Do the same for:

\[
R_D
\]

This becomes a **multi-channel reciprocal RF signature**.

---

# 12. First Important Plot

Generate:

## Mean Reciprocal RSSI vs Wi-Fi Channel

Compare:

- Air
- Empty plastic
- 25% water
- 50% water
- 75% water
- 100% water

The objective is to determine whether different material states produce repeatable channel-response patterns.

---

# 13. Phase 5 — Position Invariance

Move the test material to different positions along the RF path.

Example:

```text
10 cm from A
25 cm
50 cm
75 cm
90 cm from A
```

Research question:

> Is the system learning the material state, or simply learning object position?

A servo motor can automate this step and improve repeatability.

---

# 14. Phase 6 — Hardware Perturbation Test

This is one of the most important experiments in the project.

Train models under normal conditions.

Then deliberately introduce hardware variation.

Possible perturbations:

- Slight temperature increase
- Safe voltage variation
- Different USB supply
- ESP8266 orientation change
- Long operating duration
- Cold-start versus warmed-up board

Compare the following approaches.

## Model A — Raw RSSI

\[
R_{AB}
\]

## Model B — Bidirectional RSSI

\[
R_{AB}, R_{BA}
\]

## Model C — Reciprocal Features

\[
R_C, R_D
\]

## Model D — Full Hardware-Aware Model

\[
R_C, R_D, T, V, Clock
\]

The key result should be whether Model D remains stable when the hardware state changes.

---

# 15. Hardware Compensation Model

Start with an interpretable model.

\[
R_{corrected}
=
R_C
-
\alpha(T_A-T_{A0})
-
\beta(T_B-T_{B0})
-
\gamma(V_A-V_{A0})
-
\delta(V_B-V_{B0})
\]

Later add clock terms:

\[
R^*
=
R_C
-
\alpha \Delta T_A
-
\beta \Delta T_B
-
\gamma \Delta V_A
-
\delta \Delta V_B
-
\lambda \Delta C_A
-
\mu \Delta C_B
\]

where:

- \(R^*\): corrected reciprocal RSSI
- \(\Delta T\): temperature change from baseline
- \(\Delta V\): voltage change from baseline
- \(\Delta C\): clock-drift change from baseline

Estimate the coefficients experimentally.

---

# 16. Data Processing Pipeline

```text
Raw packet logs
       |
       v
Data cleaning
       |
       v
Outlier detection
       |
       v
Window aggregation
       |
       +-- Mean RSSI
       +-- Median RSSI
       +-- Standard deviation
       +-- Packet loss
       +-- Rc
       +-- Rd
       |
       v
Hardware compensation
       |
       v
Feature extraction
       |
       v
Classification / Regression
```

---

# 17. Machine-Learning Plan

Do not start with deep learning.

## Baselines
- Logistic Regression
- Linear Regression

## Classical Models
- Random Forest
- Support Vector Machine
- Gradient Boosting
- XGBoost / LightGBM if required

## Optional Later Models
- Multilayer Perceptron
- 1D CNN

The first objective is to determine whether the proposed features work, not to maximize model complexity.

---

# 18. Research Task A — Classification

Possible classes:

```text
Air
Plastic
Dry material
Wet material
Water-filled container
```

Metrics:

- Accuracy
- Precision
- Recall
- F1 score
- Confusion matrix

---

# 19. Research Task B — Regression

Predict continuous or ordinal material state.

Example:

```text
Water fill level:
0%
25%
50%
75%
100%
```

Metrics:

- MAE
- RMSE
- \(R^2\)

Regression may provide a stronger result than simple classification.

---

# 20. Later Material Experiments

Once the first controlled experiment works, expand to:

```text
Dry wood / wet wood
Dry cloth / wet cloth
Dry soil / moist soil
Water / saline water
Empty pipe / water-filled pipe
```

Do not begin with too many material classes.

Establish a stable methodology first.

---

# 21. Experimental Repetition

For each condition:

\[
N \ge 20
\]

independent trials are recommended.

An independent trial should involve:

```text
Remove object
Reset position
Replace object
Measure again
```

Do not treat hundreds of consecutive packets as hundreds of independent experiments.

---

# 22. Dataset Splitting

Do not randomly split packets from the same experiment into training and test sets.

Use trial-level splitting.

Example:

```text
Trials 1–15  -> Training
Trials 16–18 -> Validation
Trials 19–20 -> Testing
```

Even better:

```text
Day 1–3 -> Training
Day 4   -> Testing
```

This evaluates cross-day robustness.

---

# 23. Strongest Evaluation

Train under normal conditions:

```text
25 °C
Stable USB supply
Fixed board orientation
```

Test under unseen conditions:

```text
35 °C
Different USB supply
Minor orientation variation
```

Compare:

\[
RSSI_{raw}
\]

against:

\[
RSSI_{corrected}
\]

The central goal is to show that hardware-aware reciprocal sensing performs more consistently under drift.

---

# 24. Ablation Study

The paper should contain an ablation study.

| Method | Features |
|---|---|
| Baseline 1 | RSSI A → B |
| Baseline 2 | RSSI A → B + B → A |
| Proposed A | \(R_C, R_D\) |
| Proposed B | \(R_C, R_D, T\) |
| Proposed C | \(R_C, R_D, T, V\) |
| Full Model | \(R_C, R_D, T, V, Clock\) |

This identifies which sensing dimensions actually contribute to robustness.

---

# 25. Statistical Analysis

Do not rely only on ML accuracy.

Use:

- Mean
- Median
- Standard deviation
- Confidence intervals
- Pearson correlation
- Spearman correlation
- ANOVA when assumptions are satisfied
- Kruskal-Wallis where appropriate

Investigate relationships such as:

\[
corr(T, RSSI)
\]

\[
corr(V, RSSI)
\]

\[
corr(ClockDrift, RSSI)
\]

---

# 26. Software Repository Structure

```text
reciprosense/
|
+-- firmware/
|   +-- nodeA/
|   +-- nodeB/
|
+-- controller/
|   +-- experiment.py
|   +-- channel_scheduler.py
|   +-- serial_logger.py
|
+-- analysis/
|   +-- preprocessing.py
|   +-- features.py
|   +-- calibration.py
|   +-- statistics.py
|   +-- plots.py
|   +-- train.py
|
+-- data/
|   +-- raw/
|   +-- processed/
|
+-- configs/
|
+-- docs/
|
+-- README.md
```

Every experiment should receive a unique identifier.

Example:

```text
EXP_001
EXP_002
EXP_003
```

Never overwrite raw experimental data.

---

# 27. Recommended Dataset Schema

```text
experiment_id
trial_id
timestamp
tx_node
rx_node
channel
packet_id
rssi
temperature_tx
temperature_rx
voltage_tx
voltage_rx
current_tx
current_rx
clock_offset_tx
clock_offset_rx
material
position_cm
fill_percentage
ambient_temperature
notes
```

---

# 28. Research Paper Structure

## 1. Introduction
Explain:

- RSSI sensing is inexpensive
- RSSI is affected by hardware instability
- Existing sensing systems often focus on propagation changes
- Proposed approach uses bidirectional measurements and hardware telemetry

## 2. Related Work
Review:

- RSSI sensing
- Wi-Fi sensing
- CSI sensing
- Material sensing
- Channel reciprocity
- Oscillator drift
- Hardware impairment compensation

## 3. System Design
Describe:

\[
R_{AB}
\]

\[
R_{BA}
\]

\[
R_C
\]

\[
R_D
\]

## 4. Hardware-Aware Model
Introduce:

- Temperature
- Voltage/current
- Clock drift
- Compensation equations

## 5. Experimental Setup
Document:

- Distance
- Orientation
- Packet count
- Channel
- Material
- Position
- Environmental conditions

## 6. Results
Present:

- Raw RSSI
- Reciprocal measurements
- Multi-channel signatures
- Hardware drift
- Classification
- Regression

## 7. Ablation Study
Compare feature combinations.

## 8. Discussion
Cover:

- Wi-Fi interference
- Channel overlap
- Antenna orientation
- Environmental variability
- Hardware limitations
- Generalization limits

## 9. Conclusion
Summarize the feasibility of hardware-aware reciprocal Wi-Fi sensing using extremely low-cost commodity radios.

---

# 29. Main Novelty Target

The project should **not** be positioned primarily as:

> "ESP8266 detects water."

Instead, target:

> **Hardware-induced RSSI variation can be measured and partially separated from propagation-induced variation using reciprocal measurements and low-cost hardware telemetry.**

Material sensing is the validation workload.

---

# 30. Development Roadmap

```text
STEP 1
ESP A <-> ESP B packet exchange
        |
        v
STEP 2
Log bidirectional RSSI
        |
        v
STEP 3
Add temperature / voltage / timing telemetry
        |
        v
STEP 4
Characterize hardware drift
        |
        v
STEP 5
Create reciprocal features Rc and Rd
        |
        v
STEP 6
Introduce controlled materials
        |
        v
STEP 7
Add multi-channel measurements
        |
        v
STEP 8
Build the experimental dataset
        |
        v
STEP 9
Train baseline models
        |
        v
STEP 10
Introduce deliberate hardware disturbances
        |
        v
STEP 11
Compare corrected vs uncorrected sensing
        |
        v
STEP 12
Perform ablation and statistical analysis
        |
        v
STEP 13
Release dataset + firmware + analysis code
        |
        v
STEP 14
Prepare research paper
```

---

# 31. First Milestone

The first milestone should answer:

\[
\boxed{
Can we repeatedly measure R_{AB} \neq R_{BA}
\text{ and explain part of the difference?}
}
\]

If yes, proceed to hardware characterization.

The second milestone is:

\[
\boxed{
Does an inserted material affect R_C
\text{ differently from }
R_D?
}
\]

The third milestone is:

\[
\boxed{
Does hardware-aware compensation improve
cross-day or cross-condition sensing robustness?
}
\]

If the third result is strongly positive, it becomes the core contribution of the paper.

---

# 32. Success Criteria

The project should be considered successful if it demonstrates at least one of the following:

1. Reciprocal features outperform one-direction RSSI under hardware drift.
2. Temperature/voltage/timing compensation improves cross-condition accuracy.
3. Multi-channel reciprocal signatures distinguish controlled material states.
4. Corrected RSSI produces significantly lower variance across days or devices.
5. The method remains effective using only very low-cost commodity hardware.

---

# 33. Final Research Positioning

**Working title:**

> **ReciproSense: Hardware-Aware Reciprocal Wi-Fi Sensing Using Commodity ESP8266 Nodes**

Alternative:

> **Beyond RSSI: Hardware-Aware Reciprocal RF Sensing on Ultra-Low-Cost ESP8266 Devices**

The key story is:

```text
Two cheap Wi-Fi radios
        +
Bidirectional measurements
        +
Hardware telemetry
        +
Clock reference
        +
Multi-channel observations
        |
        v
Separating radio hardware drift
from environmental propagation changes
```

That is the research direction to protect throughout the project.
