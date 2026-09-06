<!-- Source audit: generated from the uploaded Python research implementation. :contentReference[oaicite:0]{index=0} -->

# HALP-Bench: Pre-Generation Hallucination Probing for Vision-Language Models

> **Research implementation for supervised, pre-generation hallucination probing on HALP-Bench, optimized for Google Colab Free and NVIDIA T4-class hardware.**

This project extracts internal representations from vision-language models (VLMs) **before token generation** and trains lightweight supervised probes to predict model-specific hallucination annotations. The current implementation is deliberately scoped to two T4-compatible VLMs and a deterministic cohort of up to 8,000 benchmark rows per model.

The central experimental constraint is strict: **the VLM is not asked to generate an answer during representation extraction**. The model receives the image and the original benchmark question, and the probe operates on internal representations produced by that multimodal forward pass.

## Abstract

Vision-language models can produce factually incorrect or visually unsupported answers, yet their internal states may contain information predictive of such failures before any output token is generated. This implementation studies that pre-generation signal using a HALP-style probing protocol.

For each model, the pipeline:

1. resolves an existing local HALP-Bench copy without replacing the benchmark;
2. joins model-specific manually reviewed hallucination labels using `question_id`;
3. creates a leakage-resistant grouped train/validation/test split using `image_name` as the grouping variable;
4. extracts three representation families from a single multimodal forward pass:

   * visual features (`VF`);
   * vision-token hidden states (`VT`);
   * query-token hidden states (`QT`);
5. trains a linear logistic-regression baseline and compact multilayer perceptrons (MLPs);
6. selects probe configurations using validation data only;
7. refits the selected configuration on train + validation data;
8. evaluates the locked model on the test split and reports discrimination, classification, calibration, and bootstrap uncertainty measures.

The implementation prioritizes **provenance, leakage control, resumability, memory safety, and refusal to fabricate labels or metrics** over aggressive automation.

## Scientific Scope

This repository is a **capped, two-model research implementation**, not a claim of reproducing the complete eight-model HALP study.

The configured research scope is:

| Component               | Current implementation                         |
| ----------------------- | ---------------------------------------------- |
| Benchmark cohort        | Up to 8,000 rows per active model              |
| Active VLMs             | SmolVLM2-2.2B-Instruct; Qwen2.5-VL-3B-Instruct |
| Extraction              | Pre-generation multimodal forward pass         |
| Representation families | VF, VT, QT                                     |
| Selected decoder layers | Early, quarter, middle, three-quarter, final   |
| Probe baselines         | Logistic regression; compact MLP               |
| Probe seeds             | 42, 52, 62                                     |
| Split grouping          | `image_name`                                   |
| Default probe device    | CPU                                            |
| Feature storage         | HDF5, float16                                  |
| Test uncertainty        | Percentile bootstrap AUROC, 1,000 replicates   |

The `8,000`-row setting is an explicit operational cap. It should not be described as a full 10,000-row benchmark evaluation.

## Research Question

The implementation asks whether information already present in a VLM's internal representation is predictive of a **model-specific, manually reviewed hallucination label** before generation.

Let an image-question pair be denoted by $x=(I,q)$, with binary reviewed target $y\in{0,1}$. A representation extractor maps the pair to a vector $h(x)\in\mathbb{R}^d$. A probe then estimates the probability of hallucination from that representation.

The study is therefore a **representation-probing experiment**, not a generative answer-quality evaluation. The probe does not generate text and is not treated as an independent semantic judge.

## Scientific Background

### Pre-generation probing

The defining methodological choice is to stop the VLM at the multimodal forward-pass stage. The source code explicitly avoids calling `generate()` during feature extraction.

This matters because the signal being studied is available **prior to emitted answer tokens**. The experimental target is therefore closer to:

> *Can the model's internal state predict whether the model will be judged hallucinating?*

than to:

> *Can a separate model classify a generated answer as hallucinating?*

The latter is not what this implementation evaluates.

### Model-specific supervision

The public benchmark metadata does not itself supply the model-specific supervised endpoint used here. The pipeline therefore requires the corresponding manually reviewed CSV for each active model and refuses to infer hallucination labels from `gt_answer`, the question text, or unrelated columns.

The supervised target is:

* `1`: the imported model-specific reviewed annotation marks the example as hallucinated;
* `0`: the imported model-specific reviewed annotation marks the example as non-hallucinated.

The expected reviewed field is `is_hallucinating_manual`. The automatic field `is_hallucinating` is treated as a diagnostic comparison when present, not as a silent fallback for a manually reviewed source.

### Leakage control

HALP-Bench can contain multiple questions associated with the same image. A row-wise random split can therefore place different questions from the same image into both training and evaluation partitions.

To avoid this, `image_name` is used as `group_id`, and train, validation, and test partitions are required to have pairwise-disjoint image groups.

The split procedure uses seeded `GroupShuffleSplit` trials and chooses a valid partition whose class prevalence is close to the full experimental cohort while enforcing the presence of both classes in every partition.

## Methodology

### End-to-end pipeline

```text
Existing HALP-Bench copy
        |
        v
Benchmark metadata + reviewed model-specific labels
        |
        v
One-to-one join on question_id
        |
        v
Image-group identifier = image_name
        |
        v
Fixed deterministic cohort (up to 8,000 rows/model)
        |
        v
Group-aware train / validation / test split
        |
        v
Image + original question
        |
        v
Single VLM multimodal forward pass
        |
        +-------------------+
        |                   |
        v                   v
       VF              Decoder hidden states
                            |
                      +-----+-----+
                      |           |
                      v           v
                     VT          QT
                      \           /
                       \         /
                        v       v
                      Standardization
                            |
                +-----------+-----------+
                |                       |
                v                       v
        Logistic regression          MLP sweep
                |                       |
                +-----------+-----------+
                            |
                            v
                   Validation selection
                            |
                            v
                  Train + validation refit
                            |
                            v
                     Locked test set
                            |
                            v
         Metrics + bootstrap AUROC + artifacts
```

### 1. Dataset resolution and integrity

The code searches for an existing HALP-Bench directory under Google Drive and, if necessary, an existing local `HALP-Bench.zip`. It does not enable benchmark downloading.

The expected benchmark file is:

```text
sampled_10k_relational_dataset.csv
```

The expected public-schema fields include:

```text
question_id
image_name
question
gt_answer
category
description
has_image
dataset
is_relational
```

Images are indexed once and the manifest is cached. Duplicate image basenames that resolve to different files are treated as an integrity error rather than silently choosing one.

### 2. Supervised label acquisition

For the two active models, the source code uses the following official HALP repository files:

```text
FInal_CSV_Hallucination/qwen25vl_manually_reviewed.csv
FInal_CSV_Hallucination/smolvlm_manually_reviewed.csv
```

They are downloaded only when the corresponding cached local files are unavailable. These files supply the supervised endpoint; they do not replace or modify the benchmark itself.

The join is one-to-one by `question_id`. Duplicate question IDs on the label side are rejected.

A minimum label coverage of 95% is required. Unresolved rows may be removed only after this coverage gate passes.

### 3. Deterministic cohort construction

The default experiment uses:

```python
RUN_MODE = "full"
MAX_SAMPLES = 8000
SAMPLE_SELECTION_SEED = 42
```

When the source contains more rows than the cap, exactly 8,000 rows are sampled with `random_state=42`. The selected cohort is shared by the representation-extraction and supervised-probe stages.

`smoke_test` mode is intended for structural validation rather than final benchmark reporting.

### 4. VLMs

The active registry contains:

| Model key  | Checkpoint                             | Extraction dtype |
| ---------- | -------------------------------------- | ---------------- |
| `smolvlm2` | `HuggingFaceTB/SmolVLM2-2.2B-Instruct` | `float16`        |
| `qwen25vl` | `Qwen/Qwen2.5-VL-3B-Instruct`          | `float16`        |

The Qwen configuration intentionally uses the 3B checkpoint for the T4-oriented profile rather than the larger 7B checkpoint.

The model is loaded through Hugging Face Transformers and its native processor. The extraction path uses `add_generation_prompt=False`, and no answer-generation call is made while collecting research representations.

### 5. Representation families

#### Visual feature (`VF`)

`VF` is computed from the raw visual encoder representation **before the multimodal connector/merger**. For both supported VLM families, the implementation mean-pools the visual encoder sequence into a single vector.

The representation is therefore intended to capture image-derived information before it has been transformed into the multimodal decoder stream.

#### Vision-token representation (`VT`)

`VT` is the hidden state at the final identifiable image/vision-token position of the multimodal sequence, evaluated at selected decoder layers.

The extraction code verifies that an image-token position can actually be identified. It does not fabricate a position when the model interface does not expose one cleanly.

#### Query-token representation (`QT`)

`QT` is the hidden state at the final valid token position of the pre-generation multimodal input sequence, evaluated at the same decoder layers.

The implementation determines the position from the attention mask when available and otherwise uses the final non-padding position according to the processor's tokenization.

### 6. Layer sampling

For a decoder with $L$ layers, the implementation follows a 1-based HALP layer convention:

```math
\ell_{\mathrm{early}} = 1,\qquad
\ell_{\mathrm{quarter}} = \left\lfloor 0.25L \right\rfloor,\qquad
\ell_{\mathrm{middle}} = \left\lfloor 0.50L \right\rfloor,
```

```math
\ell_{\mathrm{three\text{-}quarter}} =
\left\lfloor 0.75L \right\rfloor,\qquad
\ell_{\mathrm{final}} = L.
```

Each fractional index is clamped to the valid range $[1,L]$. Internally, Qwen decoder hooks convert the one-based convention to the corresponding zero-based module index.

The selected layer labels are:

```text
early
quarter
middle
three_quarter
final
```

### 7. Feature storage

Extracted representations are stored per model in HDF5. Feature arrays are stored as `float16`; they are converted to `float32` when read for probe training.

Typical feature keys include:

```text
vf
vt_early
vt_quarter
vt_middle
vt_three_quarter
vt_final
qt_early
qt_quarter
qt_middle
qt_three_quarter
qt_final
```

The HDF5 record also preserves metadata such as `question_id`, image name, status, and the experiment/run identifiers.

Completed samples are not recomputed during a compatible resumed run. Extraction checkpoints are written periodically, with a default interval of 500 samples.

## Mathematical Details

### Binary probing model

For an extracted representation $h_i\in\mathbb{R}^d$, the probe estimates:

```math
p_i
=
P(y_i=1\mid h_i)
=
\sigma(f_\theta(h_i)),
\qquad
\sigma(z)=\frac{1}{1+e^{-z}}.
```

For the linear baseline:

```math
f_\theta(h)=w^\top h+b.
```

The MLP replaces the linear score with a nonlinear feed-forward mapping.

### Feature standardization

For each feature coordinate $j$, the normalization statistics are calculated from the training partition only:

```math
\tilde h_{ij}
=
\frac{h_{ij}-\mu_j}
{\max(s_j,10^{-6})}.
```

The same transformation is then applied to validation and test data. During the final refit, the statistics are recomputed using **train + validation only**, and the test set remains untouched.

### MLP architecture

The MLP is a three-hidden-layer network:

```text
Linear(input_dim -> hidden1)
ReLU
BatchNorm1d
Dropout
Linear(hidden1 -> hidden2)
ReLU
BatchNorm1d
Dropout
Linear(hidden2 -> hidden3)
ReLU
BatchNorm1d
Dropout
Linear(hidden3 -> 1)
```

The scalar output is interpreted as a logit. Training uses `BCEWithLogitsLoss`.

The candidate configurations are:

| Configuration     | Hidden widths   | Learning rate | Weight decay | Dropout | Max epochs | Patience |
| ----------------- | --------------- | ------------: | -----------: | ------: | ---------: | -------: |
| `mlp_small`       | 128 → 64 → 32   |          1e-4 |         1e-4 |     0.0 |         20 |        4 |
| `mlp_base`        | 256 → 128 → 64  |          5e-4 |         1e-4 |     0.2 |         25 |        5 |
| `mlp_regularized` | 512 → 256 → 128 |          1e-3 |         1e-3 |     0.3 |         25 |        5 |

Optimization uses AdamW.

### Class-imbalance handling

The training split is inspected before probe fitting. When the larger class is at least 1.5 times the smaller class, imbalance correction is enabled.

For the MLP, the implementation uses `BCEWithLogitsLoss(pos_weight=...)` with:

```math
\alpha=\frac{n_0}{n_1},
```

where $n_0$ and $n_1$ are the training counts of classes 0 and 1.

For logistic regression, the corresponding mode uses scikit-learn's `class_weight="balanced"`.

### Validation-based model selection

MLP configurations are compared using validation AUROC. Early stopping also monitors validation AUROC.

After the best configuration is identified, the selected MLP is refit for the recorded best epoch count on train + validation data.

The validation model is released before the final refit, and normalization statistics are recomputed from train + validation only.

### Threshold selection

The default probability threshold is not blindly assumed to be optimal. For the final classifier, the validation probabilities define a candidate threshold set containing:

* 0;
* 0.5;
* 1;
* midpoints between consecutive unique validation probabilities.

The selected threshold maximizes validation F1, with balanced accuracy as the next tie-break criterion. Closeness to 0.5 is then used as a further tie-break.

The locked threshold is applied to the untouched test set.

### Evaluation metrics

The test stage reports, when mathematically defined:

* AUROC;
* average precision;
* accuracy;
* balanced accuracy;
* precision;
* recall;
* F1;
* Matthews correlation coefficient;
* Brier score;
* specificity;
* expected calibration error.

The Brier score is:

```math
\mathrm{Brier}
=
\frac{1}{N}
\sum_{i=1}^{N}(p_i-y_i)^2.
```

Expected calibration error uses ten fixed equal-width probability bins:

```math
\mathrm{ECE}
=
\sum_{b=1}^{B}
\frac{|S_b|}{N}
\left|
\operatorname{acc}(S_b)
-
\operatorname{conf}(S_b)
\right|.
```

### Multi-seed aggregation

Probe training uses seeds:

```python
SEEDS = [42, 52, 62]
```

For a metric $s$ observed across $K$ seeds, the source reports the arithmetic mean and sample standard deviation:

```math
\bar{s}
=
\frac{1}{K}
\sum_{k=1}^{K}s_k,
\qquad
s_{\mathrm{std}}
=
\sqrt{
\frac{1}{K-1}
\sum_{k=1}^{K}
(s_k-\bar{s})^2
}.
```

### Bootstrap confidence intervals

AUROC uncertainty is estimated with a percentile bootstrap using 1,000 resamples by default.

For each bootstrap replicate, test rows are sampled with replacement. Replicates containing only one class are discarded because AUROC is undefined for a single-class sample.

The reported 95% percentile interval is:

```math
\mathrm{CI}_{95\%}
=
\left[
Q_{0.025},
Q_{0.975}
\right],
```

where the quantiles are taken over valid bootstrap AUROC estimates.

The implementation requires at least `max(100, 0.10 × n_boot)` valid bootstrap replicates before presenting an interval.

## Implementation Details

### Runtime organization

The source is organized as a Colab-oriented research pipeline with explicit stages for:

1. runtime bootstrap and dependency verification;
2. Google Drive mounting and persistent project structure;
3. dataset discovery;
4. benchmark integrity auditing;
5. model-specific reviewed-label acquisition;
6. label/schema preparation;
7. grouped train/validation/test construction;
8. model loading and API smoke testing;
9. representation extraction;
10. HDF5 feature persistence and resumable checkpoints;
11. supervised probe training;
12. statistical evaluation;
13. error analysis and visualization;
14. final artifact validation;
15. archival.

### Dependency stack

The bootstrap code declares these package ranges:

| Package       | Declared range  |
| ------------- | --------------- |
| Python        | ≥ 3.10          |
| NumPy         | ≥ 2.2, < 2.6    |
| SciPy         | ≥ 1.16, < 1.17  |
| scikit-learn  | ≥ 1.9, < 2.0    |
| pandas        | ≥ 2.2, < 3.0    |
| Matplotlib    | ≥ 3.9, < 4.0    |
| seaborn       | ≥ 0.13, < 1.0   |
| h5py          | ≥ 3.12, < 4.0   |
| Transformers  | ≥ 5.0, < 6.0    |
| Accelerate    | ≥ 1.0, < 2.0    |
| Datasets      | ≥ 3.0, < 6.0    |
| tqdm          | ≥ 4.67, < 5.0   |
| num2words     | ≥ 0.5.14, < 1.0 |
| sentencepiece | ≥ 0.2.0, < 1.0  |

The bootstrap stage intentionally does not replace the Colab-provided PyTorch installation.

For Qwen extraction, `qwen-vl-utils` is installed only when it is missing.

### T4 memory and runtime safeguards

The implementation targets a 16-GB-class NVIDIA T4 environment and enforces a runtime CUDA safety ceiling of 13.5 GiB, with a preferred peak target of 13.0 GiB.

Additional safeguards include:

* one VLM kept resident at a time;
* CPU-resident probe training;
* float16 feature storage;
* adaptive Qwen visual-token clamping between 128 and 256 target visual tokens;
* reuse of Qwen raw visual features from the same multimodal forward pass;
* CUDA memory telemetry and garbage collection;
* bounded retries for numerical/runtime issues;
* atomic run-state writes;
* resumable extraction;
* checkpointing every 500 samples.
