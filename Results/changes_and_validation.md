# changes_and_validation.md

## Section 1 — Original notebook audit

- The original notebook contained 59 cells and was already structured as a HALP-style pre-generation representation pipeline.
- Its existing local-data logic pointed to an existing Drive copy of HALP-Bench rather than requiring a fresh dataset download.
- The embedded prior execution showed a real SmolVLM2 extraction smoke run on 32 HALP-Bench rows, but no supervised probe result because model-specific reviewed labels were not available in that run.
- Correct cells were preserved unless a concrete issue was identified.

## Section 2 — Changed cells

### Cell 1 — Research configuration
- Criticality score: **850/1000** (highly important)
- Change: Made full-mode/resumable defaults explicit; added model, seed, label, and imbalance controls.

### Cell 3 — Dependencies
- Criticality score: **650/1000** (important)
- Change: Removed blanket package upgrades and added the model-specific Qwen vision utility without touching the CUDA/PyTorch stack.

### Cell 6 — Drive/project paths
- Criticality score: **700/1000** (important)
- Change: Separated Google Drive mount, HALP-Bench root, metadata, images, results, checkpoints, cache, temporary, and archive paths.

### Cell 8 — Dataset resolution
- Criticality score: **950/1000** (critical)
- Change: Reused the existing Drive copy only, ranked candidate roots, and disabled any dataset re-download path.

### Cell 11 — Dataset/image audit
- Criticality score: **900/1000** (critical)
- Change: Added one-time cached image manifest, missing/duplicate checks, and a reproducible dataset fingerprint.

### Cell 15 — Reviewed-label discovery
- Criticality score: **1000/1000** (critical)
- Change: Restricted label import to explicit manually reviewed/model-specific artifacts and recorded provenance; labels are never inferred from gt_answer.

### Cell 17 — Model registry
- Criticality score: **900/1000** (critical)
- Change: Changed Qwen2.5-VL from 7B to the 3B checkpoint for the stated T4 envelope.

### Cell 20 — Label alignment
- Criticality score: **980/1000** (critical)
- Change: Added strict one-to-one question-ID joins, coverage thresholds, class checks, and explicit target semantics.

### Cell 22 — Data splitting
- Criticality score: **1000/1000** (critical)
- Change: Used deterministic image-group-aware splitting and saved partition membership for leakage auditing.

### Cell 24 — Model adapters/layer convention
- Criticality score: **920/1000** (critical)
- Change: Added current Qwen auto-class compatibility, T4-safe model metadata, and HALP layer positions {1, floor(L/4), floor(L/2), floor(3L/4), L}.

### Cell 26 — T4 smoke validation
- Criticality score: **980/1000** (critical)
- Change: Tightened VRAM headroom and enforced one-VLM-at-a-time loading with explicit cleanup.

### Cell 28 — Representation validation
- Criticality score: **980/1000** (critical)
- Change: Loaded models on demand and validated VF/VT/QT without keeping multiple VLMs resident; VF uses raw vision-encoder outputs before multimodal connector/merger.

### Cell 30 — Run identity
- Criticality score: **930/1000** (critical)
- Change: Added matching-run signatures so interrupted runs resume safely instead of mixing incompatible outputs.

### Cell 31 — Checkpoint utilities
- Criticality score: **900/1000** (critical)
- Change: Made checkpoint replacement atomic with os.replace and retained persisted progress metadata.

### Cell 32 — MLP DataLoader
- Criticality score: **900/1000** (critical)
- Change: Qwen final refit could create a singleton last batch (for example 6337 % 32 == 1), which crashes BatchNorm1d in training mode. Added a BatchNorm-safe effective batch size that shrinks only when the requested size would leave exactly one sample, while preserving every sample and the existing model architecture. Fixes the observed Qwen MLP refit failure without dropping data or changing the network. Restores Qwen MLP test evaluation; negligible additional compute/memory.

### Cell 33 — Feature extraction
- Criticality score: **980/1000** (critical)
- Change: Loaded models on demand, checkpointed per sample, recovered completion from HDF5, and released each VLM in a finally block.

### Cell 34 — Final refit standardization
- Criticality score: **930/1000** (critical)
- Change: The final refit reused statistics from train only even after validation data became part of the fit set. Recompute normalization statistics on train + validation only for the final refit, then transform test with those statistics. Uses all permitted training information while keeping test isolated. Can improve final generalization modestly; extra CPU pass is negligible for probe-sized matrices.

### Cell 34 — Validation-model release
- Criticality score: **930/1000** (critical)
- Change: The validation-selected MLP remained allocated while the final refit model was created. Release the selected model before final refit. No scientific change. Slightly reduces peak host memory during refit.

### Cell 36 — Scientific formulation
- Criticality score: **720/1000** (important)
- Change: Explicitly documented the pre-generation HALP-style setting, binary target semantics, sigmoid probe output, and validation-only threshold selection.

### Cell 38 — Probe search space
- Criticality score: **860/1000** (highly important)
- Change: Added multiple hidden sizes, learning rates, regularization/dropout settings, and multiple seeds without a combinatorial explosion.

### Cell 39 — Probe training/metrics
- Criticality score: **970/1000** (critical)
- Change: Added conditional class weighting, AUROC/AUPRC/F1/precision/recall/specificity/balanced accuracy/Brier/MCC, and validation-only threshold selection.

### Cell 41 — Experiment loop
- Criticality score: **980/1000** (critical)
- Change: Saved complete hyperparameter sweeps, selected configurations, locked validation thresholds, and test predictions.

### Cell 43 — Statistical summary
- Criticality score: **820/1000** (highly important)
- Change: Added specificity to aggregate summaries and generated a machine-readable research summary.

### Cell 47 — Error analysis
- Criticality score: **740/1000** (important)
- Change: Expanded analysis across genuine available model/representation/seed test predictions and added an error-type summary.

### Cell 49 — Figures
- Criticality score: **700/1000** (important)
- Change: Added source/class distributions, AUROC/AUPRC comparisons, layer-wise plots, sweep plots, and post-save file checks.

### Cell 50 — Diagnostics
- Criticality score: **900/1000** (critical)
- Change: Made confusion matrices use the validation-selected threshold stored with test predictions.

### Cell 52 — Run metadata
- Criticality score: **820/1000** (highly important)
- Change: Recorded dataset root, benchmark CSV, image directory, fingerprint, version information, and split artifacts.

### Cell 54 — Artifacts/changelog
- Criticality score: **760/1000** (important)
- Change: Generated manifest/README/changelog artifacts and made the generated README research-summary section idempotent on rerun.

### Cell 56 — Final validation
- Criticality score: **950/1000** (critical)
- Change: Validated CSV reloadability, figures, saved split leakage, and only marked the run complete when every requested model has a genuine test observation.

### Cell 58 — Final archive/download
- Criticality score: **850/1000** (highly important)
- Change: Created a timestamped archive, excluded dataset/model payloads, verified ZIP contents, and triggered one Colab download of the validated artifact snapshot; scientific completeness is reported separately via run_complete.json.

## Section 3 — Unchanged cells

- All cells not listed above were intentionally left unchanged because the audit did not identify a concrete correctness, reproducibility, memory, or usability defect requiring modification.

## Section 4 — Criticality score

- Scores reflect scientific impact, not the number of edited lines.

## Section 5 — Validation performed

- Dependency code path reviewed: **PASS**
- GPU/T4 checks present: **PASS**
- Google Drive persistence paths present: **PASS**
- Existing HALP-Bench-only resolution (no dataset download): **PASS**
- Cached image manifest and path integrity checks present: **PASS**
- Image-group split and zero-overlap assertions present: **PASS**
- Model-specific reviewed-label provenance required: **PASS**
- Validation-only threshold selection: **PASS**
- Hyperparameter selection kept off the test set: **PASS**
- Atomic extraction checkpoint writes: **PASS**
- Figure/table validation code present: **PASS**

### Static review passes
1. Syntax / obvious execution hazards
2. Variable dependency and ordering
3. Label construction and leakage
4. GPU / VRAM risk
5. Drive paths and persistence
6. Output-saving correctness
7. Scientific metrics and formulas
8. Re-run safety and checkpointing
9. Archive contents and exclusions

## Section 6 — Experiments actually completed

- Current run result records available: **462** completed records.
  - model=qwen25vl representation=qt_early probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.7785755197604429
  - model=qwen25vl representation=qt_early probe=logistic seed=42 split=test status=COMPLETED AUROC=0.7910350046849097
  - model=qwen25vl representation=qt_early probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_early probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_early probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_early probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.8366108303103992
  - model=qwen25vl representation=qt_early probe=mlp seed=42 split=test status=COMPLETED AUROC=0.8397700790428368
  - model=qwen25vl representation=qt_early probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.7785755197604429
  - model=qwen25vl representation=qt_early probe=logistic seed=52 split=test status=COMPLETED AUROC=0.7910350046849097
  - model=qwen25vl representation=qt_early probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_early probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_early probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_early probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.8291246721665646
  - model=qwen25vl representation=qt_early probe=mlp seed=52 split=test status=COMPLETED AUROC=0.8607740912476275
  - model=qwen25vl representation=qt_early probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.7785755197604429
  - model=qwen25vl representation=qt_early probe=logistic seed=62 split=test status=COMPLETED AUROC=0.7910350046849097
  - model=qwen25vl representation=qt_early probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_early probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_early probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_early probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.8273460815740025
  - model=qwen25vl representation=qt_early probe=mlp seed=62 split=test status=COMPLETED AUROC=0.8669665809768636
  - model=qwen25vl representation=qt_final probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.8293356913894108
  - model=qwen25vl representation=qt_final probe=logistic seed=42 split=test status=COMPLETED AUROC=0.8576027677005502
  - model=qwen25vl representation=qt_final probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_final probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_final probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_final probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.9103067817558808
  - model=qwen25vl representation=qt_final probe=mlp seed=42 split=test status=COMPLETED AUROC=0.8793815919083199
  - model=qwen25vl representation=qt_final probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.8293356913894108
  - model=qwen25vl representation=qt_final probe=logistic seed=52 split=test status=COMPLETED AUROC=0.8576027677005502
  - model=qwen25vl representation=qt_final probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_final probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_final probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_final probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.8999266456987248
  - model=qwen25vl representation=qt_final probe=mlp seed=52 split=test status=COMPLETED AUROC=0.8834778848232947
  - model=qwen25vl representation=qt_final probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.8293356913894108
  - model=qwen25vl representation=qt_final probe=logistic seed=62 split=test status=COMPLETED AUROC=0.8576027677005502
  - model=qwen25vl representation=qt_final probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_final probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_final probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_final probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.897605434247415
  - model=qwen25vl representation=qt_final probe=mlp seed=62 split=test status=COMPLETED AUROC=0.8896763808471277
  - model=qwen25vl representation=qt_middle probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.799632223640182
  - model=qwen25vl representation=qt_middle probe=logistic seed=42 split=test status=COMPLETED AUROC=0.8469055570237609
  - model=qwen25vl representation=qt_middle probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_middle probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_middle probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_middle probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.8827737974416431
  - model=qwen25vl representation=qt_middle probe=mlp seed=42 split=test status=COMPLETED AUROC=0.8932801576051702
  - model=qwen25vl representation=qt_middle probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.799632223640182
  - model=qwen25vl representation=qt_middle probe=logistic seed=52 split=test status=COMPLETED AUROC=0.8469055570237609
  - model=qwen25vl representation=qt_middle probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_middle probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_middle probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_middle probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.8907221881688556
  - model=qwen25vl representation=qt_middle probe=mlp seed=52 split=test status=COMPLETED AUROC=0.8905593061528481
  - model=qwen25vl representation=qt_middle probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.799632223640182
  - model=qwen25vl representation=qt_middle probe=logistic seed=62 split=test status=COMPLETED AUROC=0.8469055570237609
  - model=qwen25vl representation=qt_middle probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_middle probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_middle probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_middle probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.891566265060241
  - model=qwen25vl representation=qt_middle probe=mlp seed=62 split=test status=COMPLETED AUROC=0.9022355428489056
  - model=qwen25vl representation=qt_quarter probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.8140116763969975
  - model=qwen25vl representation=qt_quarter probe=logistic seed=42 split=test status=COMPLETED AUROC=0.8055822501982077
  - model=qwen25vl representation=qt_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_quarter probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.8659123566827779
  - model=qwen25vl representation=qt_quarter probe=mlp seed=42 split=test status=COMPLETED AUROC=0.8875982029166567
  - model=qwen25vl representation=qt_quarter probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.8140116763969975
  - model=qwen25vl representation=qt_quarter probe=logistic seed=52 split=test status=COMPLETED AUROC=0.8055822501982077
  - model=qwen25vl representation=qt_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_quarter probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.858225227850518
  - model=qwen25vl representation=qt_quarter probe=mlp seed=52 split=test status=COMPLETED AUROC=0.8403466833241237
  - model=qwen25vl representation=qt_quarter probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.8140116763969975
  - model=qwen25vl representation=qt_quarter probe=logistic seed=62 split=test status=COMPLETED AUROC=0.8055822501982077
  - model=qwen25vl representation=qt_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_quarter probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.8599535757709739
  - model=qwen25vl representation=qt_quarter probe=mlp seed=62 split=test status=COMPLETED AUROC=0.8530079523340459
  - model=qwen25vl representation=qt_three_quarter probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.8310841363787091
  - model=qwen25vl representation=qt_three_quarter probe=logistic seed=42 split=test status=COMPLETED AUROC=0.8438783845470053
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.897364269421305
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=42 split=test status=COMPLETED AUROC=0.8913221055666338
  - model=qwen25vl representation=qt_three_quarter probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.8310841363787091
  - model=qwen25vl representation=qt_three_quarter probe=logistic seed=52 split=test status=COMPLETED AUROC=0.8438783845470053
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.9027904780087824
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=52 split=test status=COMPLETED AUROC=0.8997309180020662
  - model=qwen25vl representation=qt_three_quarter probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.8310841363787091
  - model=qwen25vl representation=qt_three_quarter probe=logistic seed=62 split=test status=COMPLETED AUROC=0.8438783845470053
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.9012731493111727
  - model=qwen25vl representation=qt_three_quarter probe=mlp seed=62 split=test status=COMPLETED AUROC=0.9044338466713115
  - model=qwen25vl representation=vf probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.6561793462423504
  - model=qwen25vl representation=vf probe=logistic seed=42 split=test status=COMPLETED AUROC=0.7122264132811186
  - model=qwen25vl representation=vf probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vf probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vf probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vf probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.7690495091290935
  - model=qwen25vl representation=vf probe=mlp seed=42 split=test status=COMPLETED AUROC=0.7933624438411455
  - model=qwen25vl representation=vf probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.6561793462423504
  - model=qwen25vl representation=vf probe=logistic seed=52 split=test status=COMPLETED AUROC=0.7122264132811186
  - model=qwen25vl representation=vf probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vf probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vf probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vf probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.7659947546650321
  - model=qwen25vl representation=vf probe=mlp seed=52 split=test status=COMPLETED AUROC=0.8073511039569469
  - model=qwen25vl representation=vf probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.6561793462423504
  - model=qwen25vl representation=vf probe=logistic seed=62 split=test status=COMPLETED AUROC=0.7122264132811186
  - model=qwen25vl representation=vf probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vf probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vf probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vf probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.7976677351608268
  - model=qwen25vl representation=vf probe=mlp seed=62 split=test status=COMPLETED AUROC=0.7989513009634096
  - model=qwen25vl representation=vt_early probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.62662158224223
  - model=qwen25vl representation=vt_early probe=logistic seed=42 split=test status=COMPLETED AUROC=0.6432651418686783
  - model=qwen25vl representation=vt_early probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_early probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_early probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_early probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.7299355888943597
  - model=qwen25vl representation=vt_early probe=mlp seed=42 split=test status=COMPLETED AUROC=0.7040848809552411
  - model=qwen25vl representation=vt_early probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.62662158224223
  - model=qwen25vl representation=vt_early probe=logistic seed=52 split=test status=COMPLETED AUROC=0.6432651418686783
  - model=qwen25vl representation=vt_early probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_early probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_early probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_early probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.7315282816001286
  - model=qwen25vl representation=vt_early probe=mlp seed=52 split=test status=COMPLETED AUROC=0.7053431996732575
  - model=qwen25vl representation=vt_early probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.62662158224223
  - model=qwen25vl representation=vt_early probe=logistic seed=62 split=test status=COMPLETED AUROC=0.6432651418686783
  - model=qwen25vl representation=vt_early probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_early probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_early probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_early probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.7363566023895415
  - model=qwen25vl representation=vt_early probe=mlp seed=62 split=test status=COMPLETED AUROC=0.7386931504216419
  - model=qwen25vl representation=vt_final probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.6477436015957072
  - model=qwen25vl representation=vt_final probe=logistic seed=42 split=test status=COMPLETED AUROC=0.6058249044999159
  - model=qwen25vl representation=vt_final probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_final probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_final probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_final probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.707296240843273
  - model=qwen25vl representation=vt_final probe=mlp seed=42 split=test status=COMPLETED AUROC=0.6548572904403815
  - model=qwen25vl representation=vt_final probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.6477436015957072
  - model=qwen25vl representation=vt_final probe=logistic seed=52 split=test status=COMPLETED AUROC=0.6058249044999159
  - model=qwen25vl representation=vt_final probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_final probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_final probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_final probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.7330003918928424
  - model=qwen25vl representation=vt_final probe=mlp seed=52 split=test status=COMPLETED AUROC=0.6499681666386373
  - model=qwen25vl representation=vt_final probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.6477436015957072
  - model=qwen25vl representation=vt_final probe=logistic seed=62 split=test status=COMPLETED AUROC=0.6058249044999159
  - model=qwen25vl representation=vt_final probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_final probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_final probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_final probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.7099339811288524
  - model=qwen25vl representation=vt_final probe=mlp seed=62 split=test status=COMPLETED AUROC=0.6433762583187179
  - model=qwen25vl representation=vt_middle probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.6441663233417406
  - model=qwen25vl representation=vt_middle probe=logistic seed=42 split=test status=COMPLETED AUROC=0.6801167623669606
  - model=qwen25vl representation=vt_middle probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_middle probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_middle probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_middle probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.6731412723454284
  - model=qwen25vl representation=vt_middle probe=mlp seed=42 split=test status=COMPLETED AUROC=0.7024691877087188
  - model=qwen25vl representation=vt_middle probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.6441663233417406
  - model=qwen25vl representation=vt_middle probe=logistic seed=52 split=test status=COMPLETED AUROC=0.6801167623669606
  - model=qwen25vl representation=vt_middle probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_middle probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_middle probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_middle probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.6854607755458866
  - model=qwen25vl representation=vt_middle probe=mlp seed=52 split=test status=COMPLETED AUROC=0.7384348797539821
  - model=qwen25vl representation=vt_middle probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.6441663233417406
  - model=qwen25vl representation=vt_middle probe=logistic seed=62 split=test status=COMPLETED AUROC=0.6801167623669606
  - model=qwen25vl representation=vt_middle probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_middle probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_middle probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_middle probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.6984485062853583
  - model=qwen25vl representation=vt_middle probe=mlp seed=62 split=test status=COMPLETED AUROC=0.7152175479902938
  - model=qwen25vl representation=vt_quarter probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.6246018268235578
  - model=qwen25vl representation=vt_quarter probe=logistic seed=42 split=test status=COMPLETED AUROC=0.6528181534247892
  - model=qwen25vl representation=vt_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_quarter probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.710627330003919
  - model=qwen25vl representation=vt_quarter probe=mlp seed=42 split=test status=COMPLETED AUROC=0.7255723998750689
  - model=qwen25vl representation=vt_quarter probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.6246018268235578
  - model=qwen25vl representation=vt_quarter probe=logistic seed=52 split=test status=COMPLETED AUROC=0.6528181534247892
  - model=qwen25vl representation=vt_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_quarter probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.7264688445190269
  - model=qwen25vl representation=vt_quarter probe=mlp seed=52 split=test status=COMPLETED AUROC=0.7599524301467939
  - model=qwen25vl representation=vt_quarter probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.6246018268235578
  - model=qwen25vl representation=vt_quarter probe=logistic seed=62 split=test status=COMPLETED AUROC=0.6528181534247892
  - model=qwen25vl representation=vt_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_quarter probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.7170634163007324
  - model=qwen25vl representation=vt_quarter probe=mlp seed=62 split=test status=COMPLETED AUROC=0.7327589313600653
  - model=qwen25vl representation=vt_three_quarter probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.6364239275701639
  - model=qwen25vl representation=vt_three_quarter probe=logistic seed=42 split=test status=COMPLETED AUROC=0.6584100136943516
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.6671623943647819
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=42 split=test status=COMPLETED AUROC=0.6660410109795065
  - model=qwen25vl representation=vt_three_quarter probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.6364239275701639
  - model=qwen25vl representation=vt_three_quarter probe=logistic seed=52 split=test status=COMPLETED AUROC=0.6584100136943516
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.6815619441904398
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=52 split=test status=COMPLETED AUROC=0.6839788097926627
  - model=qwen25vl representation=vt_three_quarter probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.6364239275701639
  - model=qwen25vl representation=vt_three_quarter probe=logistic seed=62 split=test status=COMPLETED AUROC=0.6584100136943516
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.6740958831154475
  - model=qwen25vl representation=vt_three_quarter probe=mlp seed=62 split=test status=COMPLETED AUROC=0.6417605650721956
  - model=smolvlm2 representation=qt_early probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.8108439576737686
  - model=smolvlm2 representation=qt_early probe=logistic seed=42 split=test status=COMPLETED AUROC=0.8161279774417252
  - model=smolvlm2 representation=qt_early probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_early probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_early probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_early probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.8622329654031545
  - model=smolvlm2 representation=qt_early probe=mlp seed=42 split=test status=COMPLETED AUROC=0.8687483809596609
  - model=smolvlm2 representation=qt_early probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.8108439576737686
  - model=smolvlm2 representation=qt_early probe=logistic seed=52 split=test status=COMPLETED AUROC=0.8161279774417252
  - model=smolvlm2 representation=qt_early probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_early probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_early probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_early probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.8639656598499756
  - model=smolvlm2 representation=qt_early probe=mlp seed=52 split=test status=COMPLETED AUROC=0.8728888775644538
  - model=smolvlm2 representation=qt_early probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.8108439576737686
  - model=smolvlm2 representation=qt_early probe=logistic seed=62 split=test status=COMPLETED AUROC=0.8161279774417252
  - model=smolvlm2 representation=qt_early probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_early probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_early probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_early probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.8656912238669746
  - model=smolvlm2 representation=qt_early probe=mlp seed=62 split=test status=COMPLETED AUROC=0.8801166983043075
  - model=smolvlm2 representation=qt_final probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.8182809959784375
  - model=smolvlm2 representation=qt_final probe=logistic seed=42 split=test status=COMPLETED AUROC=0.8956913355331049
  - model=smolvlm2 representation=qt_final probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_final probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_final probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_final probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.8745900002852173
  - model=smolvlm2 representation=qt_final probe=mlp seed=42 split=test status=COMPLETED AUROC=0.8691008540039664
  - model=smolvlm2 representation=qt_final probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.8182809959784375
  - model=smolvlm2 representation=qt_final probe=logistic seed=52 split=test status=COMPLETED AUROC=0.8956913355331049
  - model=smolvlm2 representation=qt_final probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_final probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_final probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_final probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.8756381734690966
  - model=smolvlm2 representation=qt_final probe=mlp seed=52 split=test status=COMPLETED AUROC=0.8915359756071668
  - model=smolvlm2 representation=qt_final probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.8182809959784375
  - model=smolvlm2 representation=qt_final probe=logistic seed=62 split=test status=COMPLETED AUROC=0.8956913355331049
  - model=smolvlm2 representation=qt_final probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_final probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_final probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_final probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.8778129545649013
  - model=smolvlm2 representation=qt_final probe=mlp seed=62 split=test status=COMPLETED AUROC=0.8823037298442749
  - model=smolvlm2 representation=qt_middle probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.8585429679701092
  - model=smolvlm2 representation=qt_middle probe=logistic seed=42 split=test status=COMPLETED AUROC=0.8585500193223174
  - model=smolvlm2 representation=qt_middle probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_middle probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_middle probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_middle probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.8850646016941902
  - model=smolvlm2 representation=qt_middle probe=mlp seed=42 split=test status=COMPLETED AUROC=0.8995451823729504
  - model=smolvlm2 representation=qt_middle probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.8585429679701092
  - model=smolvlm2 representation=qt_middle probe=logistic seed=52 split=test status=COMPLETED AUROC=0.8585500193223174
  - model=smolvlm2 representation=qt_middle probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_middle probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_middle probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_middle probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.8853070363081487
  - model=smolvlm2 representation=qt_middle probe=mlp seed=52 split=test status=COMPLETED AUROC=0.8991247627176946
  - model=smolvlm2 representation=qt_middle probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.8585429679701092
  - model=smolvlm2 representation=qt_middle probe=logistic seed=62 split=test status=COMPLETED AUROC=0.8585500193223174
  - model=smolvlm2 representation=qt_middle probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_middle probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_middle probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_middle probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.8814566042041014
  - model=smolvlm2 representation=qt_middle probe=mlp seed=62 split=test status=COMPLETED AUROC=0.8959185320134704
  - model=smolvlm2 representation=qt_quarter probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.85163001625738
  - model=smolvlm2 representation=qt_quarter probe=logistic seed=42 split=test status=COMPLETED AUROC=0.8144590388102549
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.8694703516727988
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=42 split=test status=COMPLETED AUROC=0.8715893986300265
  - model=smolvlm2 representation=qt_quarter probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.85163001625738
  - model=smolvlm2 representation=qt_quarter probe=logistic seed=52 split=test status=COMPLETED AUROC=0.8144590388102549
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.8698054818744474
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=52 split=test status=COMPLETED AUROC=0.8862701132585071
  - model=smolvlm2 representation=qt_quarter probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.85163001625738
  - model=smolvlm2 representation=qt_quarter probe=logistic seed=62 split=test status=COMPLETED AUROC=0.8144590388102549
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.8729785231453753
  - model=smolvlm2 representation=qt_quarter probe=mlp seed=62 split=test status=COMPLETED AUROC=0.8871279392217565
  - model=smolvlm2 representation=qt_three_quarter probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.8537406234847837
  - model=smolvlm2 representation=qt_three_quarter probe=logistic seed=42 split=test status=COMPLETED AUROC=0.8838197886011067
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.8945338124982174
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=42 split=test status=COMPLETED AUROC=0.894797412932788
  - model=smolvlm2 representation=qt_three_quarter probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.8537406234847837
  - model=smolvlm2 representation=qt_three_quarter probe=logistic seed=52 split=test status=COMPLETED AUROC=0.8838197886011067
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.8908259889906164
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=52 split=test status=COMPLETED AUROC=0.903851298841935
  - model=smolvlm2 representation=qt_three_quarter probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.8537406234847837
  - model=smolvlm2 representation=qt_three_quarter probe=logistic seed=62 split=test status=COMPLETED AUROC=0.8838197886011067
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.8859487749921565
  - model=smolvlm2 representation=qt_three_quarter probe=mlp seed=62 split=test status=COMPLETED AUROC=0.8971245843578408
  - model=smolvlm2 representation=vf probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.6735689227346624
  - model=smolvlm2 representation=vf probe=logistic seed=42 split=test status=COMPLETED AUROC=0.7085047923594037
  - model=smolvlm2 representation=vf probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vf probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vf probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vf probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.7543638230512536
  - model=smolvlm2 representation=vf probe=mlp seed=42 split=test status=COMPLETED AUROC=0.7472938138857392
  - model=smolvlm2 representation=vf probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.6735689227346624
  - model=smolvlm2 representation=vf probe=logistic seed=52 split=test status=COMPLETED AUROC=0.7085047923594037
  - model=smolvlm2 representation=vf probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vf probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vf probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vf probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.7664142494509568
  - model=smolvlm2 representation=vf probe=mlp seed=52 split=test status=COMPLETED AUROC=0.7624225514801745
  - model=smolvlm2 representation=vf probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.6735689227346624
  - model=smolvlm2 representation=vf probe=logistic seed=62 split=test status=COMPLETED AUROC=0.7085047923594037
  - model=smolvlm2 representation=vf probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vf probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vf probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vf probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.7496399132939733
  - model=smolvlm2 representation=vf probe=mlp seed=62 split=test status=COMPLETED AUROC=0.7341971046250408
  - model=smolvlm2 representation=vt_early probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.6285366931918656
  - model=smolvlm2 representation=vt_early probe=logistic seed=42 split=test status=COMPLETED AUROC=0.7038589428356669
  - model=smolvlm2 representation=vt_early probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_early probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_early probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_early probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.7641645988420183
  - model=smolvlm2 representation=vt_early probe=mlp seed=42 split=test status=COMPLETED AUROC=0.7457119318495493
  - model=smolvlm2 representation=vt_early probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.6285366931918656
  - model=smolvlm2 representation=vt_early probe=logistic seed=52 split=test status=COMPLETED AUROC=0.7038589428356669
  - model=smolvlm2 representation=vt_early probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_early probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_early probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_early probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.7640077293859274
  - model=smolvlm2 representation=vt_early probe=mlp seed=52 split=test status=COMPLETED AUROC=0.7465973611234972
  - model=smolvlm2 representation=vt_early probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.6285366931918656
  - model=smolvlm2 representation=vt_early probe=logistic seed=62 split=test status=COMPLETED AUROC=0.7038589428356669
  - model=smolvlm2 representation=vt_early probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_early probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_early probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_early probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.7655906848064802
  - model=smolvlm2 representation=vt_early probe=mlp seed=62 split=test status=COMPLETED AUROC=0.7498927717545939
  - model=smolvlm2 representation=vt_final probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.6456925073443427
  - model=smolvlm2 representation=vt_final probe=logistic seed=42 split=test status=COMPLETED AUROC=0.7013449182305004
  - model=smolvlm2 representation=vt_final probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_final probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_final probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_final probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.74864165311885
  - model=smolvlm2 representation=vt_final probe=mlp seed=42 split=test status=COMPLETED AUROC=0.7353606903375672
  - model=smolvlm2 representation=vt_final probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.6456925073443427
  - model=smolvlm2 representation=vt_final probe=logistic seed=52 split=test status=COMPLETED AUROC=0.7013449182305004
  - model=smolvlm2 representation=vt_final probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_final probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_final probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_final probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.7398141809988307
  - model=smolvlm2 representation=vt_final probe=mlp seed=52 split=test status=COMPLETED AUROC=0.728905762297275
  - model=smolvlm2 representation=vt_final probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.6456925073443427
  - model=smolvlm2 representation=vt_final probe=logistic seed=62 split=test status=COMPLETED AUROC=0.7013449182305004
  - model=smolvlm2 representation=vt_final probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_final probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_final probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_final probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.7452689598128975
  - model=smolvlm2 representation=vt_final probe=mlp seed=62 split=test status=COMPLETED AUROC=0.751043617477567
  - model=smolvlm2 representation=vt_middle probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.6266685205784204
  - model=smolvlm2 representation=vt_middle probe=logistic seed=42 split=test status=COMPLETED AUROC=0.6675584659353913
  - model=smolvlm2 representation=vt_middle probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_middle probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_middle probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_middle probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.7394220073586035
  - model=smolvlm2 representation=vt_middle probe=mlp seed=42 split=test status=COMPLETED AUROC=0.7531074957851868
  - model=smolvlm2 representation=vt_middle probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.6266685205784204
  - model=smolvlm2 representation=vt_middle probe=logistic seed=52 split=test status=COMPLETED AUROC=0.6675584659353913
  - model=smolvlm2 representation=vt_middle probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_middle probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_middle probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_middle probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.7374540087276461
  - model=smolvlm2 representation=vt_middle probe=mlp seed=52 split=test status=COMPLETED AUROC=0.7471366873479164
  - model=smolvlm2 representation=vt_middle probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.6266685205784204
  - model=smolvlm2 representation=vt_middle probe=logistic seed=62 split=test status=COMPLETED AUROC=0.6675584659353913
  - model=smolvlm2 representation=vt_middle probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_middle probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_middle probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_middle probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.7441566127606172
  - model=smolvlm2 representation=vt_middle probe=mlp seed=62 split=test status=COMPLETED AUROC=0.7253767851910361
  - model=smolvlm2 representation=vt_quarter probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.6504912866147572
  - model=smolvlm2 representation=vt_quarter probe=logistic seed=42 split=test status=COMPLETED AUROC=0.7005932588468611
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.7568737343487065
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=42 split=test status=COMPLETED AUROC=0.7547912977378026
  - model=smolvlm2 representation=vt_quarter probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.6504912866147572
  - model=smolvlm2 representation=vt_quarter probe=logistic seed=52 split=test status=COMPLETED AUROC=0.7005932588468611
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.7690275519808334
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=52 split=test status=COMPLETED AUROC=0.7502622314516368
  - model=smolvlm2 representation=vt_quarter probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.6504912866147572
  - model=smolvlm2 representation=vt_quarter probe=logistic seed=62 split=test status=COMPLETED AUROC=0.7005932588468611
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.7634515558597872
  - model=smolvlm2 representation=vt_quarter probe=mlp seed=62 split=test status=COMPLETED AUROC=0.7450728090402966
  - model=smolvlm2 representation=vt_three_quarter probe=logistic seed=42 split=validation status=COMPLETED AUROC=0.5911839365676963
  - model=smolvlm2 representation=vt_three_quarter probe=logistic seed=42 split=test status=COMPLETED AUROC=0.6349355993528086
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=42 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=42 split=validation status=COMPLETED AUROC=0.7424310487436182
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=42 split=test status=COMPLETED AUROC=0.7275128567727908
  - model=smolvlm2 representation=vt_three_quarter probe=logistic seed=52 split=validation status=COMPLETED AUROC=0.5911839365676963
  - model=smolvlm2 representation=vt_three_quarter probe=logistic seed=52 split=test status=COMPLETED AUROC=0.6349355993528086
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=52 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=52 split=validation status=COMPLETED AUROC=0.7375110521662245
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=52 split=test status=COMPLETED AUROC=0.7394969402791757
  - model=smolvlm2 representation=vt_three_quarter probe=logistic seed=62 split=validation status=COMPLETED AUROC=0.5911839365676963
  - model=smolvlm2 representation=vt_three_quarter probe=logistic seed=62 split=test status=COMPLETED AUROC=0.6349355993528086
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=62 split=validation_sweep status=COMPLETED AUROC=nan
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=62 split=validation status=COMPLETED AUROC=0.7372900088417329
  - model=smolvlm2 representation=vt_three_quarter probe=mlp seed=62 split=test status=COMPLETED AUROC=0.6977947078083395

### Embedded pre-repair execution evidence
- SmolVLM2: 32 real HALP-Bench rows were processed in the notebook's prior smoke run for structural representation validation.
- No supervised probe metric was available in that prior run because reviewed hallucination labels were not found locally.

## Section 7 — Known limitations

- This repair environment does not have access to the user's live Google Drive mount, so the corrected notebook itself cannot honestly claim a newly executed 10k-row Colab run here.
- Model weights are not included in the final research archive.
- Model-specific reviewed HALP labels must exist locally for supervised probe training.
- Architecture-specific VF/VT/QT extraction is recorded explicitly; unsupported tensors are blocked rather than silently substituted.
- Colab runtime interruptions remain possible; the extraction and probe stages are designed to resume from persistent checkpoints.

## Validation status

- Current run ID: `halp_vlm_hallucination_t4_20260831T224100Z`
- Current run root: `/content/drive/MyDrive/HALP_Bench_Project/runs/halp_vlm_hallucination_t4_20260831T224100Z`
- Final summary available: `True`
- Extraction statuses: `{"qwen25vl": "COMPLETED", "smolvlm2": "COMPLETED"}`
- Probe statuses: `{"qwen25vl": "COMPLETED", "smolvlm2": "COMPLETED"}`
