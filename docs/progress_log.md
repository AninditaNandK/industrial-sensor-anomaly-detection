# Dissertation Progress Log

## Week 1

- Loaded the SWaT process datasets for 10 November and 11 November.
- Cleaned column names and converted timestamps into datetime format.
- Created binary attack labels from the documented attack intervals.
- Verified dataset time ranges, attack distributions, and data quality.
- Grouped variables into continuous sensor values, actuator/status variables, alarm variables, and process-state variables.
- Converted continuous `.Pv` sensor variables to numeric format and handled missing values.
- Created cleaned and labelled datasets for subsequent modelling.
- Performed exploratory visualisation of representative sensor behaviour during normal and attack periods.

## Week 2

- Implemented a one-step persistence forecasting baseline for individual sensors.
- Defined prediction residuals using the absolute difference between observed and forecast sensor values.
- Developed reusable evaluation functions for row-level anomaly detection.
- Evaluated continuous process sensors independently.
- Compared residual thresholds based on different quantiles of normal operating data.
- Calculated precision, recall, F1-score, and false-alarm rate.
- Added attack-event-level evaluation rather than relying only on timestamp-level metrics.
- Measured event detection rate and detection delay.
- Identified candidate sensors for the multi-sensor detector.

## Week 3

- Developed a multi-sensor forecasting-based anomaly detector.
- Introduced sensor-specific residual normalisation and system-level anomaly scoring.
- Combined sensor-level anomaly decisions into a system-level detector.
- Investigated different voting configurations and residual thresholds.
- Revised the experimental design to prevent leakage from attack labels.
- Restricted sensor selection, model development, and threshold calibration to normal operating data.
- Created development and calibration partitions using normal pre-attack observations.
- Established a frozen evaluation procedure for attack-period assessment.
- Began evaluating transfer of the detector to the second SWaT day without recalibration.

## Week 4

- Implemented a rolling-mean forecasting baseline using previous sensor observations.
- Compared persistence and rolling-mean forecasting approaches.
- Investigated trade-offs between row-level attack recall, false alarms, event detection, and detection delay.
- Implemented Random Forest forecasting using lagged sensor observations.
- Developed a first-difference Random Forest forecasting variant.
- Applied the same leakage-free calibration strategy across forecasting approaches.
- Evaluated models using both timestamp-level and event-level criteria.
- Performed frozen cross-day transfer/sensitivity evaluation on the 10 November dataset.

## Week 5

- Investigated instability caused by extremely small residual thresholds for near-constant sensors.
- Identified threshold degeneracy where calibration residuals were zero.
- Replaced the arbitrary near-zero fallback threshold with a data-derived resolution floor.
- Re-ran the persistence detector using resolution-floored thresholds.
- Confirmed that the correction removed extreme normalised anomaly-score values while preserving detector performance.
- Expanded event evaluation beyond simple one-hit detection.
- Added event coverage, consecutive alarm behaviour, and robust event-detection criteria.
- Evaluated false-alarm burden using alarm episodes in addition to row-level false-alarm rate.
- Added uncertainty estimation for cross-day performance.

## Week 6

- Investigated temporal M-of-N filtering as a possible method for reducing false alarms.
- Evaluated whether temporal filtering could preserve event detection while improving false-alarm performance.
- Tested low-alarm sensor-subset configurations as ablation studies.
- Compared the full five-sensor detector with reduced two-sensor configurations.
- Performed onset-window sensitivity analysis to examine anomaly-score discrimination near attack onset.
- Compared the corrected persistence detector with the Random Forest forecasting detector.
- Finalised the model-selection decision.
- Selected the resolution-floored persistence detector as the primary balanced detector.
- Retained the Random Forest first-difference model as a secondary comparison.
- Treated reduced-sensor configurations as low-alarm ablations rather than the primary detector.

## Week 7

- Designed a structured explanation layer on top of the final anomaly detector.
- Created event-level explanation payloads containing observed values, predictions, residuals, normalised residual scores, process context, and leading sensor contributors.
- Generated structured evidence payloads for all evaluated attack events across both datasets.
- Developed constrained LLM instructions to prevent unsupported causal or attack-type claims.
- Explicitly prevented anomaly scores from being presented as probabilities or confidence values.
- Implemented structured JSON-schema output for LLM explanations.
- Integrated the OpenAI API as an explanation layer rather than as the anomaly detector.
- Generated LLM-assisted explanations for all 21 evaluated attack events.
- Developed an automatic grounding audit for sensor references, causal claims, attack-type claims, and misuse of process-state information.
- Completed automatic evaluation of all generated explanations.

## Week 8

- Constructed a blinded human-evaluation dataset containing rule-based and LLM-assisted explanations.
- Completed human evaluation of 42 explanations.
- Assessed groundedness, technical correctness, clarity, usefulness, appropriate uncertainty, and unsupported-content flags.
- Compared rule-based and LLM-assisted explanations using descriptive statistics and paired Wilcoxon tests.
- Analysed explanation results across development and cross-day events.
- Compared automatic grounding-audit outcomes with human judgement.
- Identified the limitation that automatic rule-based audits can pass explanations that human reviewers still regard as containing unsupported content.
- Consolidated detector results, event-level metrics, explanation outputs, automatic evaluation, and human-evaluation results.
- Created final consistency and reproducibility checks.
- Removed obsolete experimental and API-generation cells from the notebook.
- Implemented resume-safe loading of previously generated LLM outputs.
- Restarted the Jupyter kernel and successfully executed the complete notebook from top to bottom without execution errors.
- Finalised the technical modelling and evaluation pipeline for dissertation writing and repository submission.
