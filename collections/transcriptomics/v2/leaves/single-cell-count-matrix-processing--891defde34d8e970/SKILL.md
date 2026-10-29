---
name: single-cell-count-matrix-processing
description: Use when a raw cells × genes scRNA-seq count matrix (e.g., Tabula Muris
  '00_facs_raw_data/FACS/{tissue}-counts.csv') must be converted to a numeric numpy
  array and paired with a signal-free null dataset for empirical hypothesis testing
  of embedding quality.
when_to_use_negative:
- Input is already a normalized or non-count dataset — the article notes 'EMBEDR's
  null-generation process has not been extended to non-normalized datasets'.
- Data is scATAC-seq peak-by-cell matrix requiring TSS-enrichment and peak-count filtering
  (peaks in ≤ 10 cells, cells with ≤ 200 peaks) rather than gene-marginal resampling.
- The goal is to evaluate embedding quality of a signal-carrying dataset without a
  signalless control — marginals alone do not define the comparison; both data and
  null embeddings are required.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: EMBEDR
  role: Python package whose null data sets are generated via marginal resampling;
    caches intermediate results via the EMBEDR class project_dir
  repo: https://github.com/ejohnson643/EMBEDR
- name: numpy
  role: Loads the count matrix into a float numpy array and performs per-gene resampling
    with replacement to build the null matrix
- name: scipy
  role: Required dependency of the EMBEDR package; supports empirical distribution
    handling during resampling
- name: seaborn
  role: Renders overlay histograms/KDEs comparing real and null per-gene marginal
    distributions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:b989b69ea2e5fc8a71dee474522dea0e6a9fa8f193d6db92d72f5c0c74621891
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ejohnson643/EMBEDR
  repo_url: https://github.com/ejohnson643/EMBEDR
  tools:
  - EMBEDR
  - numpy
  - scipy
  - seaborn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# single-cell-count-matrix-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Loads a raw single-cell count matrix (cells × genes, e.g., Tabula Muris FACS counts CSV) into a numpy array and processes it into the EMBEDR workflow, including constructing marginally-resampled null datasets that preserve per-gene marginals while destroying inter-gene correlations.

## When to use

Use when a raw cells × genes scRNA-seq count matrix (e.g., Tabula Muris '00_facs_raw_data/FACS/{tissue}-counts.csv') must be converted to a numeric numpy array and paired with a signal-free null dataset for empirical hypothesis testing of embedding quality.

## When NOT to use

- Input is already a normalized or non-count dataset — the article notes 'EMBEDR's null-generation process has not been extended to non-normalized datasets'.
- Data is scATAC-seq peak-by-cell matrix requiring TSS-enrichment and peak-count filtering (peaks in ≤ 10 cells, cells with ≤ 200 peaks) rather than gene-marginal resampling.
- The goal is to evaluate embedding quality of a signal-carrying dataset without a signalless control — marginals alone do not define the comparison; both data and null embeddings are required.

## Inputs

- Raw or preprocessed scRNA-seq count matrix (cells × genes), e.g., Tabula Muris FACS tissue-counts.csv
- Per-gene empirical expression distributions from the original data

## Outputs

- null_data_matrix.npy — numpy array of the same shape as the input, built by per-gene resampling with replacement
- Overlay plots of real vs. null per-gene marginal distributions (histograms/KDEs, as in Figure 3B)
- Cached intermediate results in the EMBEDR project directory (default ./projects/)

## How to apply

Load the preprocessed count matrix (e.g., Tabula Muris bone marrow FACS counts) into a numpy array of shape (cells, genes). For every gene column independently, draw expression values with replacement from that column's empirical distribution to build a null data matrix of identical shape — this marginal resampling preserves per-gene distributions while destroying the joint (correlation) structure between genes. Save the result (e.g., as null_data_matrix.npy) and cache intermediate results in a project directory, as the EMBEDR package does by default. Validate by overlaying real and null per-gene marginals (histograms/KDEs for a representative panel of genes with seaborn/matplotlib) and confirming they nearly coincide, matching Figure 3B of the paper. The null matrix feeds EMBEDR's empirical hypothesis test, which estimates whether samples are better-embedded than a DRA might do by chance.

## Related tools

- **EMBEDR** (Python package whose null data sets are generated via marginal resampling; caches intermediate results via the EMBEDR class project_dir) — https://github.com/ejohnson643/EMBEDR
- **numpy** (Loads the count matrix into a float numpy array and performs per-gene resampling with replacement to build the null matrix)
- **scipy** (Required dependency of the EMBEDR package; supports empirical distribution handling during resampling)
- **seaborn** (Renders overlay histograms/KDEs comparing real and null per-gene marginal distributions)

## Examples

```
from EMBEDR import EMBEDR; import numpy as np; X = np.loadtxt("./data/mnist2500_X.txt").astype(float); embObj = EMBEDR(project_dir='./'); embObj.fit(X)
```

## Evaluation signals

- Null matrix shape exactly equals the input count matrix shape (cells × genes preserved).
- Real and null per-gene marginal distributions visually coincide when overlaid for a representative gene panel (Figure 3B).
- Inter-gene correlations in the null matrix differ from the original data (joint distribution altered while marginals preserved).
- Intermediate results are cached and retrievable from the EMBEDR project directory so re-runs do not waste prior embeddings.
- The null dataset, when embedded, produces the lower bound of the EMBEDR p-value distribution determined by the number of nulls embedded.

## Limitations

- EMBEDR's null-generation process has not been extended to non-normalized datasets, so raw pipelines cannot yet be assessed end-to-end.
- There is a finite resolution to the calculated p values since the null distributions are calculated empirically; the number of nulls embedded determines the lower bound on p values.
- No theoretical description of the tails of the null distributions or a principled parameterization method exists.
- FIt-SNE support requires fftw to be installed; without it, the fast t-SNE implementation cannot be used.

## Evidence

data_name = f"00_facs_raw_data/FACS/{tissue.title()}-counts.csv""
