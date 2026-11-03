# RF Fence — MATLAB Forward Scatter Radar Simulation

**Current status: simulation prototype; MATLAB execution and experimental field validation remain to be documented.**

RF Fence explores a radio-frequency crossing sensor using **a separate transmitter (TX) and receiver (RX)**. This repository contains scalar diffraction models, synthetic channel and detection experiments, and a MATLAB dashboard. It contains no SDR acquisition driver or measured RF dataset.

**وجهة نظر الفريق:** المشروع حاليًا نموذج MATLAB لمحاكاة Forward Scatter Radar باستخدام مرسل TX ومستقبل RX منفصلين. نرى أن قابلية التطبيق الميداني واعدة مبدئيًا وتستحق تجربة عملية، لكنها لم تُثبت بعد. يلزم التحقق باستخدام SDR وهوائيات وقياسات حقيقية، مع تقييم الضوضاء وتعدد المسارات (multipath) ومناطق ضعف الكشف (blind spots) والإنذارات الكاذبة (false alarms) وهندسة توزيع المرسل والمستقبل (geometry). كشف عبور جسم لا يثبت أنه درون؛ تصنيف الأهداف يحتاج بيانات وتجارب مستقلة.

## 1. Idea

A target passing near the TX–RX line of sight (LOS) changes the received field through diffraction and interference with the direct field. The simulation investigates whether that change can support a crossing alarm.

The model represents the target as a **flat rectangular screen** with an opacity factor. The drone shown in the dashboard is an enlarged illustration; its body, materials and rotors are not the electromagnetic target model. A single baseline is a crossing-sensor concept, not demonstrated volumetric surveillance, localization or drone identification.

## 2. Architecture

The physical arrangement is bistatic: TX and RX occupy different positions. The implemented screen geometry places the target between them near the LOS.

```text
TX [0, 0, hAnt] ------ crossing plane x = xc ------ RX [L, 0, hAnt]
                              |
                     target motion along y
                     height = hAnt + z0

Geometry + frequency + screen + trajectory
  -> scalar scattered fraction s(t)
  -> normalized direct-path field E/E0 = 1 - s(t)
  -> synthetic channel -> received complex samples r(t)
  -> amplitude processing -> detector A or B
  -> calibrated threshold -> alarm events
  -> independent synthetic evaluation / dashboard playback
```

- **Physics:** paraxial Babinet/Fresnel closed form, numerical scalar Kirchhoff integration and an axial disk reference.
- **Coordinates:** use `fsr_scattered_field` for LOS-relative offsets. Raw `fsr_kirchhoff_numeric` takes absolute target coordinates; the baseline must be parallel to x, with the crossing strictly between TX and RX.
- **Channel:** `fsr_channel_model` adds optional reflected paths, slow gain drift, slow interference, residual carrier frequency offset (CFO) and complex Gaussian noise. All parameters are assumptions.
- **Detection:** A measures normalized amplitude deviation with a noise estimate; B uses a bank of matched templates derived from the direct-path model. Thresholds are calibrated on target-free synthetic recordings, then tested on separate recordings.
- **Dashboard:** `RF_Drone_Fence_App` calls the comparison engine through its API. It reveals precomputed samples during animation; real-time SDR processing has not been implemented.

The comparison engine uses the shared channel model. The older `fsr_detection_demo` retains its own simpler channel implementation and does not expose the reflected-path configuration. Multipath is **off by default** in the comparison and dashboard.

## 3. What has been demonstrated in simulation — evidence and limits

The repository implements a simulation workflow and checks designed to assess the following:

| Evidence available | Supported interpretation | Status |
|---|---|---|
| Diffraction equations and staged physics checks | Internal consistency, symmetry, coordinate handling, quadrature convergence and agreement between scalar models can be tested | MATLAB execution not verified in this documentation update |
| Values in `README_MATLAB_v2.txt` | Upstream reports expected values from a line-by-line NumPy transcription | Reference predictions, not independently reproduced here or measured |
| Channel tests | Identity, phase, delay, noise statistics, reproducibility and legacy compatibility can be checked | Test source exists; no new PASS claim |
| Detection experiments and comparison tests | Conditional synthetic Pd, FA/hour, latency and confidence intervals can be generated; seed separation and causality can be checked | No verified results archive is included |
| Dashboard self-test | Consistency of animation, scores, thresholds and app logic can be checked | GUI and self-test were not run here |

For example, the upstream text predicts a peak relative amplitude change of about **0.0856** for its nominal 2.4 GHz, 20 m baseline, 0.30 × 0.20 m opaque-screen case. This is a conditional model prediction, not measured sensitivity or detection probability.

**No experimentally proven detection performance is claimed.** Some inherited code comments use “verified” or “tested”; those comments are not a substitute for archived MATLAB execution logs. See [the review record](docs/REVIEW.md).

## 4. What has not been demonstrated

- Agreement with laboratory or field RF measurements, including the validity of the screen approximation for an actual drone.
- A working SDR/antenna prototype, calibrated link budget, measured range or deployable coverage.
- Measured probability of detection, false-alarm rate, latency or long-term reliability.
- Generalization from synthetic calibration to a different site, weather condition or channel.
- Classification of drones versus birds, people, vehicles or other moving objects.
- Range, bearing, altitude estimation, multiple-target tracking or continuous perimeter coverage.
- End-to-end streaming operation, acquisition latency, dropped-sample handling or processing throughput.

## 5. Feasibility assessment — team position

**Promising enough for a controlled hardware experiment; field feasibility remains unproven.** This is the team's engineering assessment of the current scope, not a measured conclusion.

The simulation offers a tractable way to explore signatures and detector settings before acquiring RF data. A useful next milestone is repeatable detection of a known obstruction crossing a controlled baseline. A claim of reliable drone detection requires substantially more evidence.

| Constraint | Consequence for feasibility | Required evidence |
|---|---|---|
| Noise and receiver behavior | Assumed CNR does not establish a physical link budget. Receiver gain changes, clipping and drift may obscure small amplitude variations | Calibrated I/Q recordings, noise measurements, fixed-gain tests and saturation checks |
| Multipath | Reflections can reinforce or cancel the field and alter the signature. The current tapped-path model is phenomenological, without site surfaces or per-path target geometry | Measurements with controlled reflectors and at multiple sites; compare observed signatures with predictions |
| Blind spots | Low-sensitivity regions can arise from diffraction structure, destructive interference, target height and antenna patterns | Map detection and missed detections over crossing position, height, direction and orientation |
| False alarms | Synthetic Gaussian noise and slow disturbances do not establish behavior under vegetation, people, birds, traffic or other interference | Long target-free recordings, frozen thresholds and independent test periods, with uncertainty intervals |
| Geometry | Current experiments use a fixed baseline and straight perpendicular crossings. Detector B assumes the configured crossing location and screen geometry | Vary baseline length, antenna placement, crossing angle/location, speed and height; test template mismatch |
| Target representation and classification | An opaque rectangle does not capture real materials, aspect changes or rotor micro-Doppler. An alarm alone does not identify a drone | Labeled recordings of actual targets and confusers, evaluated on held-out sessions |
| Model validity and deployment | Scalar physical optics and the paraxial approximation have limits. Single-baseline coverage and hardware timing remain unverified | Convergence/model checks, measurement comparisons, streaming benchmarks and coverage trials |

Additional limits: the numerical “exact” mode uses exact path lengths **within a scalar Kirchhoff approximation**; it is not a Maxwell/full-wave solution. At the default 250 Hz detector sample rate, ordinary reflected-path delays are far below one sample; this channel model is not evidence of delay-resolved multipath ranging. The normalized field does not specify TX power, antenna gain, receiver noise figure or RF bandwidth. Those require a separate measured link budget.

## 6. Hardware and testing roadmap

All stages below are planned; completing a simulation does not complete the corresponding physical experiment.

| Stage | Work | Evidence required before advancing |
|---|---|---|
| 1. Reproduce the software | Run physics, channel, detection, comparison and app checks in MATLAB; save release, configuration, commit, seeds, logs and figures | Review every failure; archive reproducible outputs and distinguish INFO entries from asserted checks |
| 2. Establish a bench link | Use an SDR TX/RX pair or equivalent RF source and SDR receiver, suitable antennas, mounts, cables and attenuation; characterize frequency offset and gain | Stable target-free I/Q, measured noise and drift, known gain settings, no clipping, documented sample rate and RF configuration |
| 3. Known-obstruction trial | Move a screen of measured dimensions through a measured baseline; record crossing time with an independent reference | Repeated signatures across positions and heights, uncertainty estimates and explicit model-versus-measurement discrepancies |
| 4. Controlled target trial | Record actual targets and confusers across aspect, speed, height and crossing angle | Separate calibration/test sessions, fixed detector settings, Pd per crossing, FA/hour, latency and confidence intervals |
| 5. Field robustness | Include target-free periods, clutter, reflectors, weather variation and different sites | Coverage/blind-spot maps, independent performance estimates and a documented failure analysis |
| 6. Streaming prototype | Add SDR acquisition, signal conditioning and streaming detection; evaluate additional baselines only if coverage evidence supports them | Measured processing latency, throughput, sample loss and end-to-end alarm behavior |

Choose hardware specifications from the bench measurements and required operating band; no particular SDR, power, range or cost is established by this repository. Classification is a later research task requiring its own dataset and evaluation.

## 7. How to run

### Requirements and setup

Use **MATLAB R2020b or newer** for the dashboard, as specified by its source. The supplied simulation uses base MATLAB functions and does not require specialized toolboxes. An eventual SDR interface may require hardware-specific support that is not included here. A supported MATLAB release still needs execution verification.

Clone this fork, or download its ZIP and extract it:

```sh
git clone https://github.com/eng-talal7/rf-fence-matlab.git
```

For MATLAB Online, upload the extracted project files to MATLAB Drive. In either desktop MATLAB or MATLAB Online, set **Current Folder** to the project folder and keep **all 12 `.m` files together**.

### Open the dashboard

```matlab
RF_Drone_Fence_App
```

The first use of a frequency/CNR combination calibrates thresholds and can take time. It caches them in `fsr_app_threshold_cache.mat`. Choose a scenario and press **Play**; switch detector A/B, frequency, CNR, speed or height as needed. All displayed signals are synthetic. A “TARGET DETECTED” display means an alarm occurred; the separate ground-truth evaluation indicates a hit, miss or false alarm.

### Run the supplied checks

Run these commands individually and inspect each summary:

```matlab
fsr_validate_physics
fsr_channel_model_tests
fsr_detection_demo('tests')
fsr_detector_comparison('tests')
RF_Drone_Fence_App('selftest')
```

The physics script starts with `clear; clc; close all;`, so save your work first. The app self-test checks logic without opening its window; it does not replace a GUI check.

### Generate synthetic results

```matlab
rDemo = fsr_detection_demo();
Rquick = fsr_detector_comparison('quick');
```

The quick mode reduces trial counts; it does not establish field performance. For the full default comparison and the dashboard's optional statistics panel:

```matlab
R = fsr_detector_comparison();
save('fsr_comparison_results.mat', 'R');
RF_Drone_Fence_App
```

Restart the dashboard or use **Load results...** to load a saved run. The panel shows protocol statistics for matching saved cells, not an estimate from the single animation. The cache and results files are generated locally and are not supplied as validated evidence.

To explore an assumed reflected path in the comparison engine:

```matlab
cfg = struct();
cfg.paths = struct('gain', -0.7, 'excess_m', 0.2237, ...
                   'modulation', 'static');
Rmp = fsr_detector_comparison(cfg);
```

These path values are illustrative assumptions, not site measurements. Changing the channel requires recalibration and independent evaluation.

## 8. Repository files

| File | Role |
|---|---|
| `RF_Drone_Fence_App.m` | Dashboard, threshold cache, optional results loading and app self-test |
| `fsr_detection_demo.m` | Standalone amplitude detector experiment and basic tests |
| `fsr_detector_comparison.m` | Detector A/B comparison, API and protocol/integration tests |
| `fsr_channel_model.m` | Synthetic noise, CFO, drift, interference and optional reflected paths |
| `fsr_channel_model_tests.m` | Channel implementation checks |
| `fsr_scattered_field.m` | Shared LOS-relative interface to the diffraction models |
| `fsr_babinet_paraxial.m` | Rectangular-screen paraxial Babinet/Fresnel model |
| `fsr_kirchhoff_numeric.m` | Numerical scalar Kirchhoff integration |
| `fsr_disk_axial.m` | Axial opaque-disk reference |
| `fsr_fresnel_cs.m` | Fresnel C/S rational approximations |
| `fsr_wavelength.m` | Frequency-to-wavelength conversion |
| `fsr_validate_physics.m` | Staged physics consistency, convergence and sensitivity checks |
| `README_MATLAB_v2.txt` | Preserved upstream physics reference; its seven-file setup describes the older physics-only subset |
| `docs/REVIEW.md` | Source provenance, file-review scope and verification status |

## 9. Project context and technical notes

This is the team's RF Fence project repository. A copy of the project files was hosted by a teammate before being copied into this account; that hosting arrangement does not define individual authorship or contributions. The MATLAB project files and reference text are retained.

The Fresnel routine retains its attribution to Stephen L. Moshier's Cephes implementation. Its comment refers to `README_MATLAB.txt`, which is absent upstream; the supplied reference file is `README_MATLAB_v2.txt`. No new code license is assigned by this documentation update.

For terminology, [MathWorks' bistatic radar documentation](https://www.mathworks.com/help/radar/bistatic-radar.html) describes separate TX/RX systems. Its [SDR diagnostics documentation](https://www.mathworks.com/help/comm/rtlsdrradio-diagnostics.html) provides frequency-offset measurement context. These references are background; they do not validate this implementation or its field feasibility.
