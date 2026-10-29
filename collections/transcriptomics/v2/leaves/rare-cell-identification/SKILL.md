---
name: rare-cell-identification
description: Use when you need per-cell rareness scores from a large scRNA-seq expression
  matrix (tens of thousands of cells) and clustering-based rare cell tools (RaceID,
  GiniClust) are too slow or memory inefficient.
when_to_use_negative:
- When the goal is to separate technical outliers from genuine minor cell types —
  FiRE does not discriminate between an outlier and cells representing minor cell
  types.
- When doublet detection is required — doublet rates vary between 1% and 30%, a range
  that does not necessarily comply with the notion of cell rareness.
- When input is already cluster labels or a small dataset where established tools
  like RaceID or GiniClust suffice; FiRE's advantage is scaling to tens of thousands
  of profiles.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3316
tools:
- name: FiRE
  role: 'Core algorithm: LSH-based rareness scoring of each expression profile (L=100,
    M=50, H=1017881); installed via ./INSTALL --py'
  repo: github.com/princethewinner/FiRE
- name: Python
  role: Runtime environment and package interface (python setup.py build) used to
    install and run FiRE and its preprocessing utilities
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:b73a82484e7911daeba4122e34bc91d3cc9259ae253476ee0b7f20cb7e7debda
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# rare-cell-identification

## Summary

Implements FiRE (Finder of Rare Entities) rareness scoring: sketch each cell's expression profile into M=50-bit locality-sensitive hash codes with L=100 estimators and use bucket populousness to assign a per-cell rareness score without clustering. Enables rare cell detection on ultra-large scRNA-seq data in seconds (e.g. ~31 s for ~68k profiles).

## When to use

Use when you need per-cell rareness scores from a large scRNA-seq expression matrix (tens of thousands of cells) and clustering-based rare cell tools (RaceID, GiniClust) are too slow or memory inefficient.

## When NOT to use

- When the goal is to separate technical outliers from genuine minor cell types — FiRE does not discriminate between an outlier and cells representing minor cell types.
- When doublet detection is required — doublet rates vary between 1% and 30%, a range that does not necessarily comply with the notion of cell rareness.
- When input is already cluster labels or a small dataset where established tools like RaceID or GiniClust suffice; FiRE's advantage is scaling to tens of thousands of profiles.

## Inputs

- scRNA-seq expression matrix (np.array [nCells, nGenes], rows = cells, columns = genes), e.g. GSE94820 293T-Jurkat data
- Gene names vector (np.array [nGenes]) for ranger_preprocess variable-gene selection
- FiRE python module built from github.com/princewinner/FiRE (Cython extension against boost >= 1.54.0)

## Outputs

- Per-cell FiRE rareness scores (np.array [nCells], higher = rarer), e.g. written to fire_scores.csv with cell barcode and FiRE score columns
- Optional binary rare/abundant predictions after IQR-based thresholding (score >= Q3 + 1.5*IQR)
- Optionally the preprocessed log2-transformed matrix of 1000 variable genes (np.array [nCells, nVariableGenes])

## How to apply

Obtain the scRNA-seq expression matrix (e.g. 293T-Jurkat data from GEO accession GSE94820, Zheng et al.), with rows as cells and columns as genes. Preprocess by selecting the top 1000 variable genes with log2 transformation via ranger_preprocess (utils/preprocess.py). Install FiRE in the python environment with the repository's INSTALL script in --py mode, which builds the Cython extension against the boost library (>= 1.54.0, only boost/random.hpp is needed). Create the model with its fixed architecture — FiRE.FiRE(L=100, M=50, H=1017881, seed=5489) — fit it on the preprocessed matrix, and compute scores with model.score(); the consensus score combines L bucket-populousness density estimates as -2*sum_l log(p_il), where higher values denote rare cells. Optionally convert scores to binary predictions using IQR-based thresholding (threshold = Q3 + 1.5*IQR). Write per-cell scores to fire_scores.csv with cell barcode and FiRE score columns.

## Related tools

- **FiRE** (Core algorithm: LSH-based rareness scoring of each expression profile (L=100, M=50, H=1017881); installed via ./INSTALL --py) — github.com/princethewinner/FiRE
- **Python** (Runtime environment and package interface (python setup.py build) used to install and run FiRE and its preprocessing utilities)

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489, verbose=0); model.fit(preprocessedData); score = np.array(model.score(preprocessedData))
```

## Evaluation signals

- fire_scores.csv exists and contains exactly one score row per cell in the input expression matrix.
- All FiRE scores are finite numeric values; higher scores mark rare cells.
- The IQR rule (score >= Q3 + 1.5*IQR) yields a small subset of predicted rare cells, consistent with the expected minor subpopulation.
- On the two-species Jurkat/293T benchmark, the planted rare cells should be recovered in the high-score subset (as in example/jurkat_simulation.py).
- FiRE should complete in seconds, e.g. ~1.1 s scoring the 1580-cell preprocessed demo data or ~31 s for ~68k profiles.

## Limitations

- FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types, and outlier cells can get submerged into minor cell clusters when rare cells are subsequently clustered with dropClust.
- Doublet rate may vary between 1% and 30%, a wide range that does not necessarily comply with the notion of cell rareness, limiting FiRE's application to doublet detection.
- While RaceID and GiniClust offer binary predictions, FiRE gives a rareness score to every profile — a secondary thresholding step (IQR rule) is needed for binary calls.

## Evidence

- [abstract] FiRE assigns rareness score per cell: "an algorithm that, in a matter of seconds, assigns a rareness score to every individual expression profile under study"
- [discussion] FiRE avoids clustering: "Most notable among these is its ability to avoid clustering as an intermediate step."
- [readme] Model parameters and score semantics: [evidence span withheld — anchor status: external_doc]
- [readme] Preprocessing selects 1000 variable genes: [evidence span withheld — anchor status: external_doc]
- [methods] Install command: "--py                       : python        : Install FiRE in python environment."
