---
name: locality-sensitive-hashing-density-estimation
description: Use when a monolithic, clustering-free estimate of per-cell rareness/density
  is needed on ultra-large scRNA-seq data (tens of thousands of profiles) where pairwise-clustering
  methods like RaceID or GiniClust scale too slowly.
when_to_use_negative:
- The goal is doublet detection — doublet rate varies between 1% and 30%, which does
  not necessarily comply with the notion of cell rareness
- The task requires distinguishing true outliers from cells of minor cell types —
  FiRE does not discriminate between an outlier and cells representing minor cell
  types
- The input is a small dataset where slower but more interpretable clustering-based
  methods (RaceID, GiniClust) suffice
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: FiRE
  role: 'Core LSH-based rareness scorer: sketches each expression profile into an
    M=50-bit hash code and scores bucket populousness as density'
  repo: github.com/princethewinner/FiRE
- name: Python
  role: Runtime environment in which the FiRE Cython module is installed via ./INSTALL
    --py and used to score cells
- name: dropClust
  role: Optional downstream clustering of FiRE-flagged rare cells for annotation;
    administers no special treatment to outliers
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:999e736ea2189a8a81963cd4bf885abf043c657c1a74acafedf188ca106af4f0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  - dropClust
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# locality-sensitive-hashing-density-estimation

## Summary

Sketches each cell's expression profile into an M=50-bit hash code using L=100 locality-sensitive hash estimators, and uses bucket populousness as a local density proxy so that rare cells receive high FiRE rareness scores (consensus score -2*sum_l log(p_il)).

## When to use

Use when a monolithic, clustering-free estimate of per-cell rareness/density is needed on ultra-large scRNA-seq data (tens of thousands of profiles) where pairwise-clustering methods like RaceID or GiniClust scale too slowly.

## When NOT to use

- The goal is doublet detection — doublet rate varies between 1% and 30%, which does not necessarily comply with the notion of cell rareness
- The task requires distinguishing true outliers from cells of minor cell types — FiRE does not discriminate between an outlier and cells representing minor cell types
- The input is a small dataset where slower but more interpretable clustering-based methods (RaceID, GiniClust) suffice

## Inputs

- scRNA-seq expression matrix (rows = cells, columns = genes), e.g. GSE94820 293T-Jurkat data
- Gene names array (np.array [nGenes]) passed to ranger_preprocess
- Preprocessed log2-transformed matrix of top-1000 variable genes (np.array [nCells, nVariableGenes])

## Outputs

- Per-cell FiRE rareness scores (np.array[nCells]), higher = rarer
- fire_scores.csv with cell barcode and FiRE score columns
- Binary rare-cell predictions via IQR thresholding (Q3 + 1.5*IQR)
- Model internals: model.dims, model.thresholds, model.weights (L x M), model.bins (L x H x dynamic)

## How to apply

Preprocess the expression matrix (rows = cells, columns = genes) by selecting the top ~1000 variable genes via ranger_preprocess with log2 transformation. Instantiate FiRE with its fixed architecture: L=100 estimators, M=50-bit hash codes, hash table size H=1017881, seed 5489. Fit the model on the preprocessed matrix, then compute each cell's bucket-populousness density p_il per estimator l and combine into the consensus score -2*sum_l log(p_il); higher FiRE scores indicate rarer cells. For a binary rare-cell call, threshold with the IQR rule (score >= Q3 + 1.5*IQR). FiRE avoids clustering as an intermediate step, which is what makes it fast (~31 s for ~68k profiles).

## Related tools

- **FiRE** (Core LSH-based rareness scorer: sketches each expression profile into an M=50-bit hash code and scores bucket populousness as density) — github.com/princethewinner/FiRE
- **Python** (Runtime environment in which the FiRE Cython module is installed via ./INSTALL --py and used to score cells)
- **dropClust** (Optional downstream clustering of FiRE-flagged rare cells for annotation; administers no special treatment to outliers)

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489, verbose=0); model.fit(preprocessedData); score = np.array(model.score(preprocessedData))
```

## Evaluation signals

- fire_scores.csv exists and contains exactly one score row per cell in the input expression matrix
- All FiRE scores are finite numeric values
- Higher scores correspond to known rare populations (e.g. label '2' cells in the 293T-Jurkat two-species benchmark)
- Rare cells flagged via IQR thresholding (score >= Q3 + 1.5*IQR) reproduce the paper's planted rare-cell recovery on the jurkat demo
- Model parameters match the fixed architecture: L=100, M=50, H=1017881

## Limitations

- FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types
- When FiRE-detected rare cells are clustered with dropClust, outlier cells may get submerged into the minor cell clusters
- Doublet rate may vary between 1% and 30%, which does not necessarily comply with the notion of cell rareness, limiting FiRE's application to doublet detection
- Python --inplace installation may fail uninstallation; the generated .so file may need manual removal
- The full jurkat demo data may require large amounts of RAM to load and preprocess

## Evidence

- [abstract] FiRE assigns a rareness score to every individual expression profile in a matter of seconds: "an algorithm that, in a matter of seconds, assigns a rareness score to every individual expression profile under study"
- [discussion] FiRE avoids clustering as an intermediate step: "Most notable among these is its ability to avoid clustering as an intermediate step."
- [discussion] FiRE speed on 68k profiles: "FiRE took ~31 s to analyze a scRNA-seq dataset containing ~68 k expression profiles."
- [readme] Fixed FiRE architecture (L, M, H): [evidence span withheld — anchor status: external_doc]
- [readme] IQR-based thresholding for binary rare calls: "q3 = np.percentile(score, 75)
iqr = stats.iqr(score)
th = q3 + 1.5*iqr"
- [readme] Higher FiRE score means rarer: "score : FiRE score of every cell : np.array[nCells]

Higher values of FiRE score represent rare cells."
