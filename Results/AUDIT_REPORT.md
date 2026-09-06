# AUDIT_REPORT.md

## Runtime audit status

- Run ID: `halp_vlm_hallucination_t4_20260831T224100Z`
- Run mode: `full`
- Sample cap: `8000` per active model
- Active models: `['smolvlm2', 'qwen25vl']`
- Scientific status: **PENDING — complete the full supervised run and validate final test artifacts.**

## Supervised endpoint provenance

The notebook requires the dedicated manual-review field when the source file provides one. The automatic `is_hallucinating` field is never accepted as a silent fallback for a `*_manually_reviewed.csv` source.

- smolvlm2: reviewed label column=`is_hallucinating_manual`; automatic-vs-reviewed disagreement=0.0.
- qwen25vl: reviewed label column=`is_hallucinating_manual`; automatic-vs-reviewed disagreement=0.8025.

## Experimental scope

The configured experiment evaluates a deterministic 8000-row cohort per model on Google Colab/T4-oriented settings. This is an operational compute scope and is not a full 10,000-row / eight-model reproduction of the published HALP study.

## Verification state

- Notebook JSON structure and every code cell are syntax-validated in the audit environment.
- Runtime-dependent VLM extraction/training cannot be re-executed here because the supplied bundle does not contain the dataset images, VLM weights, or the feature HDF5 stores required for the full pipeline.
- Existing prediction CSVs were independently re-scored for internal metric consistency; see the forensic audit report for details.

## Required interpretation

The existence of `run_complete.json` is evidence that a supplied Colab run completed, but it is not by itself evidence that the scientific target was defined with the correct reviewed-label column. All final claims must follow the current label contract and be regenerated when that contract changes.

