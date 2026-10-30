# Forecasting-Based Sensor Error Detection in Industrial Cyber-Physical Systems with LLM-Assisted Explanation

**MSc Data Science Dissertation · School of Computer Science · University of Birmingham · 2025–26**

> **Can forecasting errors provide useful evidence of abnormal industrial sensor behaviour — and can an LLM explain that evidence without overstating what it proves?**
> This project forecasts the expected next value of industrial process sensors, treats the absolute forecasting residual as anomaly evidence, and calibrates thresholds using normal operating data only. A separate explanation layer converts detector evidence into natural language and is evaluated automatically and through blinded human review.

## What This Project Does

Industrial cyber-physical systems continuously generate sensor measurements describing the physical state of a process. Abnormal readings may arise from cyberattacks, equipment faults, sensor faults, process disturbances or normal operating changes.

Rather than training a classifier to recognise known attack signatures, this project asks a different question:

> **Is the current sensor reading substantially different from what its recent normal behaviour predicts?**

The study uses **SWaT A9 industrial water-treatment data** from 10 and 11 November 2022. Persistence forecasting, rolling-mean forecasting, Random Forest level forecasting and Random Forest first-difference forecasting were investigated.

The final methodology was rebuilt to avoid attack-label leakage. Sensor selection and residual-threshold calibration use only the normal period before the first documented 11-November attack.

The second part of the project investigates explainability. A constrained LLM and a deterministic rule-based baseline receive the same structured detector evidence. Their explanations are then compared using automatic grounding checks and blinded human evaluation.

The LLM is therefore **not the anomaly detector**. It is a separately evaluated explanation layer.

---

## Headline Findings

| Finding | Result |
|---|---|
| **Leakage-free redesign reduced false alarms** | Final resolution-floored persistence achieved **13.67% FAR** on 11 November while all **17/17 documented events contained at least one alarm** |
| **Simple persistence remained competitive** | Persistence was retained as the **primary balanced detector**; RF first-difference remained the secondary ML comparison |
| **Cross-day behaviour exposed the trade-off** | Persistence: **8.25% FAR, 3/4 robust events, 14.57% coverage**; RF: **7.57% FAR, 4/4 robust events, 7.41% coverage** |
| **Timestamp and event metrics tell different stories** | 17/17 and 4/4 any-hit detection do **not** mean complete attack coverage; final timestamp recall remained low |
| **Threshold-independent ranking remained near chance** | Final normal-only persistence achieved ROC-AUC **0.488** on 11 November and **0.488** on frozen 10-November transfer; RF first-difference achieved **0.481** cross-day. Average precision was also close to attack prevalence, indicating weak timestamp-level ranking power |
| **Threshold degeneracy required correction** | `FIT101.Pv` produced a zero q=0.99 calibration threshold; a data-derived **0.000320 resolution floor** corrected the score scaling without materially changing detector performance |
| **LLM explanations were not automatically better** | Rule-based explanations scored higher on groundedness, correctness, clarity and usefulness; the LLM scored higher only on appropriate uncertainty |
| **Automatic grounding was insufficient** | **21/21** LLM explanations passed predefined automatic checks, but human review still flagged unsupported content in **11/21** |

---

## Two Layers

The detection and explanation components are deliberately separated.

### Detection Layer

Forecast normal sensor behaviour and convert forecasting error into anomaly evidence.

```text
SWaT continuous .Pv sensors
        ↓
Normal-only development period
        ↓
Sensor predictability analysis
        ↓
★ Five sensors selected without attack-label performance ★
        ↓
One-step forecast

prediction(t) = actual(t−1)

        ↓
Absolute forecasting residual

residual(t) = |actual(t) − prediction(t)|

        ↓
Sensor-specific q = 0.99 threshold
learned from normal calibration data
        ↓
★ Data-derived resolution floor for zero thresholds ★
        ↓
normalised residual = residual / threshold
        ↓
Sensor triggers when normalised residual > 1
        ↓
System anomaly score =
maximum threshold-normalised residual
        ↓
Timestamp-level + attack-event-level evaluation
```

The anomaly score is a **threshold-normalised residual**, not a probability or confidence percentage.

### Explanation Layer

Translate detector evidence into natural language without allowing the LLM to perform detection.

```text
Detected event
        ↓
Structured evidence payload

event ID
event timing
detection timestamp
detection delay
observed value
predicted value
absolute residual
normalised residual
triggered sensors
leading contributors
process-state context

        ↓
        ├── Deterministic rule-based explanation
        │
        └── Constrained gpt-5-mini explanation
                ↓
           JSON schema
           grounding rules
           uncertainty requirement
           no attack/root-cause invention
                ↓
Automatic grounding audit
                ↓
Blinded human evaluation
```

Process state is included as **contextual metadata only**. It is not used to calculate the final anomaly score.

---

## Build Pipeline

```text
Two SWaT A9 datasets
11-Nov-2022_0900_1143.csv
10-Nov-2022_1100_1200.csv
        ↓
Clean column names
parse timestamps
convert .Pv channels to numeric
handle missing observations
        ↓
Create documented binary attack labels

0 = normal
1 = attack
        ↓
Separate variable groups

31 continuous .Pv sensors
32 actuator / status variables
15 alarm variables
6 process-state variables
        ↓
Identify 3,060 normal rows
before first 11-Nov attack
        ↓
70% normal development = 2,142 rows
30% normal calibration = 918 rows
        ↓
Normal-only sensor selection
based on predictability + process-stage diversity
        ↓
Final sensors

FIT201.Pv
FIT101.Pv
AIT302.Pv
AIT402.Pv
AIT502.Pv
        ↓
Compare forecasting approaches

Persistence
Rolling mean
Random Forest — levels
Random Forest — first differences
        ↓
Residual-based anomaly detection
        ↓
q = 0.99 normal calibration thresholds
        ↓
Resolution-floor correction
        ↓
Timestamp + event-level evaluation
        ↓
Freeze models and thresholds
        ↓
Apply unchanged to 10 November
NO cross-day recalibration
        ↓
Structured event evidence
        ↓
Rule-based + LLM explanations
        ↓
Automatic audit + blinded human evaluation
```

The 10-November experiment is described as a **frozen cross-day transfer / sensitivity evaluation**, not prospective validation, because 10 November precedes the 11-November development period.

---

## Repository Structure

```text
axn582/
├── README.md
├── requirements.txt
│
├── notebooks/
│   └── 01_swat_sensor_error_detection_pipeline.ipynb
│       # complete preprocessing, forecasting, detection,
│       # robustness, explanation and evaluation pipeline
│
├── results/
│   │
│   │── DETECTOR SELECTION
│   ├── final_detector_decision.csv
│   ├── final_threshold_correction_comparison.csv
│   ├── cross_day_uncertainty_intervals_10nov.csv
│   ├── floored_persistence_event_quality_10nov.csv
│   ├── floored_persistence_event_quality_summary_10nov.csv
│   │
│   │── EXPLANATION
│   ├── all_event_explanation_payloads.jsonl
│   ├── final_llm_explanations.jsonl
│   ├── llm_grounding_audit.csv
│   ├── automatic_llm_evaluation_summary.csv
│   ├── automatic_vs_human_grounding_comparison.csv
│   ├── human_explanation_evaluation_completed.csv
│   ├── human_explanation_evaluation_completed_with_key.csv
│   ├── human_explanation_evaluation_summary.csv
│   ├── compact_human_explanation_comparison.csv
│   ├── paired_human_explanation_comparison.csv
│   ├── human_hallucination_summary.csv
│   ├── human_evaluation_by_dataset_split.csv
│   └── final_explanation_evaluation.csv
│
├── figures/
│   └── detector and evaluation visualisations
│
└── docs/
    └── progress_log.md
```

The original SWaT datasets and locally generated cleaned datasets are **not committed to the repository**.

The final notebook contains consistency checks confirming:

- one selected primary detector;
- 21 structured event payloads;
- 21 completed LLM explanations;
- 21 automatic grounding audits;
- 42 completed human explanation reviews;
- all required final output files.

---

## Results

### Development Day — 11 November 2022

Final evaluation begins at **09:51:00** after normal-only development and calibration.

| Model | Precision | Recall | F1 | FAR | Any-hit Events | Mean Delay |
|---|---:|---:|---:|---:|---:|---:|
| **Resolution-floored persistence** | **0.448** | **0.148** | **0.223** | **13.67%** | **17/17** | **15.82 s** |
| RF first-difference q=0.99 | 0.396 | 0.107 | 0.169 | 12.27% | 17/17 | 17.71 s |

Persistence was retained because it provided the stronger overall balance of recall, F1, event sensitivity and detection delay, while remaining simple and interpretable.

Earlier exploratory experiments demonstrated why recall alone was not an adequate optimisation target:

| Exploratory Model | Recall | F1 | FAR |
|---|---:|---:|---:|
| Early persistence | 0.530 | 0.434 | 38.12% |
| Rolling mean | 0.612 | 0.468 | 41.98% |
| Random Forest — levels | 0.892 | 0.589 | **85.04%** |

The Random Forest level model produced very high attack recall, but an approximately 85% false-alarm rate makes that operating point impractical.

These exploratory results are retained to document model-development decisions; they are not treated as the final leakage-free comparison.

---

### Threshold Degeneracy and Resolution Floor

The q=0.99 calibration residual for `FIT101.Pv` was zero.

Using an arbitrary tiny replacement such as `1e-9` would produce extremely large threshold-normalised anomaly scores. The final pipeline therefore defines the sensor-resolution floor using the **smallest positive residual observed in earlier normal development data**.

| Sensor | Raw q=.99 Threshold | Resolution Floor | Applied Threshold |
|---|---:|---:|---:|
| FIT201.Pv | 0.330428 | 0.000128 | 0.330428 |
| **FIT101.Pv** | **0.000000** | **0.000320** | **0.000320** |
| AIT302.Pv | 0.845135 | 0.025600 | 0.845135 |
| AIT402.Pv | 0.205078 | 0.025621 | 0.205078 |
| AIT502.Pv | 0.281983 | 0.025624 | 0.281983 |

Effect on 11-November persistence:

| Version | Precision | Recall | F1 | FAR | Events | Delay |
|---|---:|---:|---:|---:|---:|---:|
| Original q=.99 | 0.448 | 0.148 | 0.223 | 13.70% | 17/17 | 15.82 s |
| **Resolution-floored** | **0.448** | **0.148** | **0.223** | **13.67%** | **17/17** | **15.82 s** |
| Remove degenerate sensors | 0.391 | 0.030 | 0.056 | 3.54% | 16/17 | 40.56 s |

The floor corrected score scaling while essentially preserving detection performance. Removing the affected sensors lowered false alarms but severely reduced useful sensitivity.

---

### Frozen Cross-Day Transfer — 10 November 2022

All 11-November model parameters and thresholds are frozen and applied to 10 November **without recalibration**.

| Configuration | Precision | Recall | F1 | FAR | Any-hit | Robust | Delay | Coverage |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| **Persistence — primary** | **0.469** | **0.116** | **0.186** | 8.25% | 4/4 | 3/4 | **29.5 s** | **14.57%** |
| RF first-difference — comparator | 0.353 | 0.066 | 0.111 | **7.57%** | 4/4 | **4/4** | 33.5 s | 7.41% |
| Persistence — 2-sensor ablation | 0.485 | 0.024 | 0.045 | 1.59% | 4/4 | 1/4 | 70.0 s | 2.39% |
| RF — 2-sensor ablation | 0.473 | 0.019 | 0.036 | **1.31%** | 4/4 | 2/4 | 92.5 s | 1.89% |

Neither full detector dominates every metric.

**Persistence** provides:

- higher timestamp recall;
- higher F1;
- approximately twice the mean event coverage;
- slightly faster first detection.

**RF first-difference** provides:

- slightly lower false-alarm rate;
- stronger robust-event count.

Persistence is therefore labelled the **primary balanced detector**, while RF first-difference remains a **secondary ML comparator**.

The two-sensor experiments are retained as **low-alarm ablations**. They demonstrate that aggressively reducing false alarms also causes severe loss of detection sensitivity.

---

### Threshold-Independent Evaluation

Operating-point metrics can appear encouraging even when the underlying anomaly score has limited ability to rank attack timestamps above normal timestamps. ROC-AUC and average precision were therefore evaluated independently of the selected q = 0.99 threshold.

| Configuration | Dataset | ROC-AUC | Average Precision | Attack Prevalence |
|---|---|---:|---:|---:|
| Normal-only persistence | 11 November | 0.488 | 0.421 | 0.428 |
| Frozen normal-only persistence | 10 November | 0.488 | 0.396 | 0.386 |
| Frozen RF first-difference | 10 November | 0.481 | 0.394 | 0.386 |

These values are close to chance-level ROC-AUC, while average precision is close to the underlying attack prevalence. The detector therefore has **weak timestamp-level ranking power**, despite producing alarms within every documented attack event at the selected operating point.

This is why the event-level results are interpreted as operational evidence rather than proof of strong attack discrimination.

---

### Cross-Day Uncertainty

Only four documented events are available on 10 November.

For 4/4 any-hit event detection:

**Exact 95% confidence interval: 39.8%–100.0%**

False-alarm uncertainty was estimated using a moving-block bootstrap:

| Detector | FAR | 95% Interval |
|---|---:|---:|
| Persistence — all sensors | 8.25% | 3.76–14.46% |
| RF difference — all sensors | 7.57% | 5.35–10.25% |
| Persistence — subset | 1.59% | 0.82–2.22% |
| RF difference — subset | 1.31% | 0.68–1.90% |

These are treated as **cross-day uncertainty intervals**, not proof of general population-level performance.

---

### Explanation Evaluation

The explanation dataset contains:

- **21 event evidence payloads**
- **21 LLM-assisted explanations**
- **21 deterministic rule-based explanations**
- **42 blinded human evaluations**

Human ratings use a 1–5 scale.

| Criterion | Rule-Based | LLM-Assisted | LLM − Rule | Wilcoxon *p* |
|---|---:|---:|---:|---:|
| Groundedness | **2.810** | 2.381 | −0.429 | 0.2997 |
| Technical correctness | **2.952** | 2.571 | −0.381 | 0.2010 |
| Clarity | **3.476** | 2.524 | −0.952 | **0.0052** |
| Usefulness | **3.381** | 2.714 | −0.667 | 0.0791 |
| Appropriate uncertainty | 2.762 | **3.286** | +0.524 | 0.2483 |

The deterministic explanation scored higher on:

- groundedness;
- technical correctness;
- clarity;
- usefulness.

The LLM-assisted explanation scored higher only on **appropriate uncertainty**.

The clearest paired difference was clarity, favouring the rule-based explanation.

The Wilcoxon analysis is treated as **exploratory**, because the experiment contains only 21 paired events and one human reviewer.

---

### Automatic Audit vs Human Review

The automatic audit checked predefined machine-testable constraints including:

- complete output schema;
- valid confidence label;
- unknown sensor references;
- unsupported causal claims;
- unsupported attack-type claims;
- incorrect claims that process state influenced scoring.

| Measure | Count | Percentage |
|---|---:|---:|
| LLM explanations evaluated | 21 | 100.0% |
| Passed automatic grounding audit | 21 | 100.0% |
| Human review found no unsupported content | 10 | 47.62% |
| Human flagged unsupported content | **11** | **52.38%** |
| Automatic pass + human flag | **11** | **52.38%** |

The central finding is:

> **Passing predefined automatic checks does not establish complete semantic groundedness.**

Automatic rules can verify the failure modes they were designed to recognise, but human judgement can identify broader unsupported interpretations.

---

## Research Questions

1. **Can forecasting residuals calibrated using normal operating data provide useful evidence of abnormal sensor behaviour at a manageable false-alarm rate?**

2. **How do persistence, rolling-mean and Random Forest forecasting approaches compare when evaluated at both timestamp and attack-event level?**

3. **How do normal-only sensor selection, threshold calibration, threshold-degeneracy correction and temporal filtering affect the sensitivity / false-alarm trade-off?**

4. **How well does a detector developed on one SWaT operating period transfer to another operating day when models and thresholds are frozen without recalibration?**

5. **Can an LLM translate structured detector evidence into useful and appropriately cautious natural-language explanations, and how does its quality compare with deterministic rule-based explanations under automatic and human evaluation?**

RQ4 is specifically a **frozen cross-day transfer / sensitivity evaluation**, not prospective validation.

---

## Evaluation

Detection is evaluated at several complementary levels.

| Metric | What It Measures |
|---|---|
| Precision | Of all detector alarms, how many occurred during labelled attack periods |
| Timestamp recall | Proportion of labelled attack timestamps that were flagged |
| F1 | Harmonic balance between precision and recall |
| False-alarm rate | Proportion of normal timestamps incorrectly flagged |
| Any-hit event detection | Whether an event contains at least one alarm |
| ≥3-row event detection | Whether an event contains at least three alarm rows |
| Consecutive detection | Whether an event contains at least two consecutive alarm rows |
| Robust event detection | Stronger sustained/repeated evidence within an event |
| Event coverage | Alarm rows as a percentage of all rows in an event |
| Detection delay | Time from documented event start to first alarm |
| False-alarm episodes | Distinct alarm episodes during normal operation |
| ROC-AUC / average precision | Threshold-independent ranking behaviour |

Cross-day uncertainty is quantified using:

- **exact binomial intervals** for event-detection counts;
- a **moving-block bootstrap** for false-alarm rate, preserving temporal dependence in the alarm sequence.

Explanation evaluation uses:

| Criterion | Question |
|---|---|
| Groundedness | Is the explanation supported by supplied detector evidence? |
| Technical correctness | Is the technical interpretation accurate? |
| Clarity | Is the explanation easy to understand? |
| Usefulness | Would the explanation help someone inspect the anomaly? |
| Appropriate uncertainty | Does it avoid overstating what the evidence proves? |
| Unsupported-content flag | Does the explanation make an inadequately supported claim? |

---

## Tech Stack

| Tool | Purpose |
|---|---|
| Python 3 | Main implementation language |
| Jupyter Notebook | End-to-end experimental environment |
| pandas | Data loading, cleaning, timestamp handling, lag and rolling operations |
| NumPy | Vectorised residual, threshold and numerical calculations |
| scikit-learn | Random Forest regression and ML evaluation metrics |
| SciPy | Wilcoxon signed-rank testing and statistical calculations |
| Matplotlib | Sensor, anomaly-score and evaluation visualisations |
| OpenAI API — `gpt-5-mini` | Constrained structured explanation generation |
| JSON / JSONL | Structured evidence and explanation records |
| CSV | Exported model and evaluation results |
| Git / GitLab | Version control, project tracking and submission |

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://git.cs.bham.ac.uk/projects-2025-26/axn582.git
cd axn582
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Obtain the SWaT data

The original SWaT datasets are **not distributed through this repository**.

The pipeline expects authorised local copies of:

```text
11-Nov-2022_0900_1143.csv
10-Nov-2022_1100_1200.csv
```

Place them in the location expected by the notebook before running the data-preparation cells.

### 4. Open the notebook

```text
notebooks/01_swat_sensor_error_detection_pipeline.ipynb
```

Run the notebook from top to bottom.

The main detection pipeline does not require an external API.

### 5. Optional — regenerate LLM explanations

An OpenAI API key is required only if the explanation-generation stage is being regenerated.

```bash
export OPENAI_API_KEY="your-key-here"
```

The notebook is designed to load previously generated explanation outputs when available rather than automatically repeating completed API calls.

### 6. Final consistency checks

The final notebook verifies completion of:

```text
Primary detector
21 event payloads
21 LLM explanations
21 automatic grounding audits
42 human-reviewed explanations
Required final output files
```

---

## Future Work

- **Wider temporal validation** — evaluate more SWaT operating days and additional industrial control-system datasets to determine whether the final sensor set, thresholds and model ranking generalise beyond the two periods studied here.

- **Improve normal-only sensor selection** — combine normal-operation predictability with responsiveness or information-content measures so that highly predictable but nearly static sensors do not receive disproportionate preference.

- **Use additional process channels** — incorporate actuator/status and alarm channels alongside continuous `.Pv` sensors. Abnormal relationships between commanded actuator states and observed process responses may provide information that a univariate forecasting residual cannot capture.

- **Combine numerical and textual evidence** — retrieve equipment descriptions, process documentation, alarm descriptions, maintenance notes and operator logs alongside numerical detector evidence.

  A future explanation architecture could be:

  ```text
  Numerical sensor anomaly
          +
  Retrieved process / equipment documentation
          ↓
  Evidence-grounded explanation
  ```

  This would allow the system to distinguish between **what the detector observed** and **what external process documentation suggests should be inspected**.

- **Adaptive and streaming detection** — investigate online threshold adaptation, concept-drift handling and continuous monitoring under changing operating conditions.

- **Longer temporal models where justified** — investigate whether richer sequential models improve forecasting after substantially larger normal training datasets are available, rather than adding complexity solely for its own sake.

- **Stronger human explanation evaluation** — include multiple independent reviewers, industrial-control domain experts and inter-rater reliability analysis.

- **Retrieval-grounded explanation** — constrain the LLM using both detector evidence and retrieved plant documentation, then compare groundedness and usefulness with the present structured-evidence-only approach.

- **Operator-support interface** — ultimately integrate detection and explanation into a human-in-the-loop interface:

  ```text
  Alarm
    ↓
  Sensor evidence
    ↓
  Relevant process documentation
    ↓
  Cautious explanation
    ↓
  Recommended verification checks
  ```

  Such a system should support human investigation rather than act as an autonomous root-cause or cyberattack diagnosis system.

---

## References

Adepu, S. and Mathur, A. (2016). *An Investigation into the Response of a Water Treatment System to Cyber Attacks.* IEEE International Symposium on High Assurance Systems Engineering.

Inoue, J., Yamagata, Y., Chen, Y., Poskitt, C.M. and Sun, J. (2017). *Anomaly Detection for a Water Treatment System Using Unsupervised Machine Learning.* IEEE International Conference on Data Mining Workshops.

Jeffrey, N., Tan, Q. and Villar, J.R. (2023). *A Review of Anomaly Detection Strategies to Detect Threats to Cyber-Physical Systems.* Electronics, 12(15), 3283.

Liu, Q. and Paparrizos, J. (2024). *The Elephant in the Room: Towards A Reliable Time-Series Anomaly Detection Benchmark.* NeurIPS Datasets and Benchmarks Track.

Parcalabescu, L. and Frank, A. (2024). *On Measuring Faithfulness or Self-consistency of Natural Language Explanations.* Proceedings of ACL 2024.

Karami, A., Jalali, S. and Ghassemi, M. (2025). *A Comprehensive Forecasting-Based Framework for Time Series Anomaly Detection: Benchmarking on the Numenta Anomaly Benchmark.* arXiv.

---

## Author

**Anindita Nandkumar** · MSc Data Science · University of Birmingham · Supervisor: **Dr Mohammad H. (Vahid) Mamduhi**

---

## Declaration of AI Use

Generative AI was used during project development for implementation support, debugging, refactoring and drafting assistance. AI-generated suggestions were not treated as automatically correct: code was executed and inspected, experimental outputs were checked, and the final pipeline contains explicit reproducibility and consistency checks.

Final methodological choices, interpretation of experimental results and claims presented in this project were reviewed and retained by the author.

Separately, **`gpt-5-mini` is itself an experimental component of the dissertation**. It receives structured evidence generated by the anomaly detector and produces natural-language explanations that are evaluated against deterministic rule-based explanations.

These two uses of generative AI are distinct.

The LLM under evaluation:

- performs no anomaly detection;
- assigns no attack labels;
- does not calculate the anomaly score;
- does not establish that a cyberattack occurred;
- does not provide validated root-cause diagnosis.

*Submitted in partial fulfilment of the MSc Data Science degree, University of Birmingham, 2025–26.*
