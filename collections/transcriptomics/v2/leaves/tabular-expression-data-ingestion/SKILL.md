---
name: tabular-expression-data-ingestion
description: Use when feeding a raw gene/protein count matrix (TSV or CSV) into FAVA's
  VAE and the values must be log2-normalized and scaled per row to [0,1] to match
  the model's sigmoid output layer.
when_to_use_negative:
- Input is already log2-normalized and scaled to [0,1] (e.g. previously produced preprocessed_matrix.tsv)
  — re-applying row-max scaling would distort values.
- Data is provided as AnnData (.h5ad) that FAVA already accepts directly via the scverse
  ecosystem, so no TSV pre-processing step is needed.
- The goal is downstream analysis (e.g. cooking protein pairs after VAE latent-space
  correlation) rather than preparing the VAE's input matrix.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: FAVA (favapy)
  role: Tool whose pre-processing layer performs log2 normalization and row-max [0,1]
    scaling of the count matrix before VAE encoding
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Language used to implement the fava.py module including the pandas-based loading
    and scaling logic
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:04a05cf6dd7ef7546c1e1df681c8ed04cbe8c948d93e4eb2c33bca5fee231030
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - FAVA (favapy)
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# tabular-expression-data-ingestion

## Summary

Ingest raw count matrices in TSV (default) or CSV via pandas and apply FAVA's pre-processing: log2 normalization followed by row-max scaling into [0,1], matching the sigmoid output layer of the VAE.

## When to use

Use when feeding a raw gene/protein count matrix (TSV or CSV) into FAVA's VAE and the values must be log2-normalized and scaled per row to [0,1] to match the model's sigmoid output layer.

## When NOT to use

- Input is already log2-normalized and scaled to [0,1] (e.g. previously produced preprocessed_matrix.tsv) — re-applying row-max scaling would distort values.
- Data is provided as AnnData (.h5ad) that FAVA already accepts directly via the scverse ecosystem, so no TSV pre-processing step is needed.
- The goal is downstream analysis (e.g. cooking protein pairs after VAE latent-space correlation) rather than preparing the VAE's input matrix.

## Inputs

- Raw count matrix in TSV format (default) or CSV, loadable with pandas
- Command-line data file path passed to favapy

## Outputs

- preprocessed_matrix.tsv — log2-normalized, row-max-scaled matrix in [0,1]

## How to apply

Load the raw count matrix with pandas, accepting TSV (default) or CSV as the input format. Apply log2 normalization to the count values to tame the dynamic range of sparse scRNA-seq/proteomics data. Then scale every value to the [0,1] interval by dividing each row's values by that row's maximum; this row-max scaling is deliberately consistent with the 0-1 scaled sigmoid output activation of the FAVA VAE, so the input range matches what the model reconstructs. Write the result to preprocessed_matrix.tsv, the file format FAVA consumes as its command-line data file.

## Related tools

- **FAVA (favapy)** (Tool whose pre-processing layer performs log2 normalization and row-max [0,1] scaling of the count matrix before VAE encoding) — https://github.com/mikelkou/fava
- **Python** (Language used to implement the fava.py module including the pandas-based loading and scaling logic)

## Examples

```
favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- All values in preprocessed_matrix.tsv fall within [0,1]; each row's maximum equals 1.
- Row-max scaling is applied after (not before) log2 normalization — verify ordering by comparing against a manual pandas implementation.
- Values differ from the raw count matrix only by the monotonic log2 + row-max transform; ranks within each row are preserved after log2.
- preprocessed_matrix.tsv is accepted as the favapy command-line data file without format errors.
- Input file format (TSV/CSV) is parsed correctly by pandas without column shifting or header loss.

## Limitations

- Only TSV (default) and CSV tabular formats are handled in this ingestion step; other formats need conversion first.
- Row-max scaling means rows whose maximum is very small (or zero) may yield unstable or degenerate scaled values.
- The [0,1] target range is dictated by the VAE's sigmoid output layer; using this transform with a different output activation would mismatch the model's expected input distribution.

## Evidence

- [methods] Preprocessing stage: log2 + row-max scaling: [evidence span withheld — anchor status: not_found]
- [methods] Sigmoid output layer rationale: [evidence span withheld — anchor status: not_found]
- [readme] Input formats and output file: [evidence span withheld — anchor status: not_found]
- [readme] Output artifact: [evidence span withheld — anchor status: not_found]
