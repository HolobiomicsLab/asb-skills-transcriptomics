---
name: scrna-seq-rare-cell-scoring
description: Use when the goal is to detect rare cell populations in a scRNA-seq count
  matrix (thousands to ~68k+ profiles) and no annotated clusters exist yet; FiRE gives
  a per-cell continuous rareness score in seconds where RaceID/GiniClust scale poorly.
when_to_use_negative:
- 'The task is doublet detection: doublet rate varies between 1% and 30%, a range
  that does not necessarily comply with the notion of cell rareness, limiting FiRE''s
  application to doublet detection.'
- 'You need to separate technical outliers from genuine minor cell-type cells: FiRE,
  in principle, does not discriminate between an outlier and cells representing minor
  cell types.'
- The input is already a cluster label table or feature summary rather than a cells-by-genes
  expression matrix — FiRE requires per-cell expression profiles to estimate bucket
  populousness.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FiRE
  role: 'Core LSH-based rareness scorer: fits the Sketch model and assigns a rareness
    score to every individual expression profile in seconds (~31 s for ~68k profiles)'
  repo: https://github.com/princethewinner/FiRE
- name: Python
  role: Primary environment for FiRE installation (./INSTALL --py), preprocessing
    (numpy, pandas, statsmodels), and scoring the expression matrix
  repo: https://github.com/princethewinner/FiRE
- name: dropClust
  role: Clusters the FiRE-detected rare cells after IQR thresholding to resolve them
    into minor clusters (e.g. R1-R13 dendritic clusters, R10 pars tuberalis sub-type)
- name: RaceID
  role: Alternative rare-cell method offering binary predictions, used as a baseline
    comparison in F1-score evaluations on the Jurkat dilution simulations
- name: GiniClust
  role: Alternative rare-cell method offering binary predictions, used as a baseline
    comparison in F1-score evaluations on the Jurkat dilution simulations
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_003
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:ea4035111f675252a85c416ee2df6c9dd14eb9957c29502509d317eeed32fdcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princethewinner/FiRE
  tools:
  - FiRE
  - GiniClust
  - LOF
  - Python
  - R
  - RaceID
  - dropClust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# scrna-seq-rare-cell-scoring

## Summary

Assign a FiRE rareness score to every individual scRNA-seq expression profile using an LSH Sketch-based density estimator (L=100 estimators, M=50-bit hash codes, H=1017881 bins), then dichotomize cells into rare vs non-rare with the interquartile-range threshold rule — without clustering as an intermediate step.

## When to use

Use when the goal is to detect rare cell populations in a scRNA-seq count matrix (thousands to ~68k+ profiles) and no annotated clusters exist yet; FiRE gives a per-cell continuous rareness score in seconds where RaceID/GiniClust scale poorly.

## When NOT to use

- The task is doublet detection: doublet rate varies between 1% and 30%, a range that does not necessarily comply with the notion of cell rareness, limiting FiRE's application to doublet detection.
- You need to separate technical outliers from genuine minor cell-type cells: FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types.
- The input is already a cluster label table or feature summary rather than a cells-by-genes expression matrix — FiRE requires per-cell expression profiles to estimate bucket populousness.

## Inputs

- scRNA-seq expression matrix (cells x genes; e.g. Zheng et al. ~68k PBMC profiles from GEO GSE94820 or ~3200-cell 293T-Jurkat mixture)
- Gene identifiers/names array (replaced by integer indices 1..nGenes if names are unavailable)
- Optional preprocessed matrix (log2-transformed, top 1000 variable genes, e.g. data/preprocessedData_jurkat_two_species_1580.txt.gz)

## Outputs

- Per-cell FiRE rareness scores (np.array[nCells]; higher = rarer, exportable as fire_scores.csv with cell barcode and score columns)
- Binary rare/non-rare cell calls after IQR thresholding (predictions vector; e.g. 4238 rare cells on ~68k PBMC)
- Model artifacts for inspection: model.dims, model.thresholds, model.weights, model.bins

## How to apply

Load the cells-by-genes expression matrix (e.g. from GEO GSE94820) and pre-process with ranger_preprocess to retain the top 1000 most variable genes with log2 transformation. Instantiate the FiRE model with its fixed architecture — L=100 locality-sensitive hash estimators, M=50 randomly sampled features per estimator, hash table size H=1017881, seed 5489 — then fit and score the preprocessed matrix. Each cell is sketched into an M=50-bit hash code per estimator, and the per-estimator bucket populousness is combined into the consensus score -2*sum_l log(p_il); higher FiRE scores indicate rarer cells. Call rare cells with the IQR rule: threshold = Q3 + 1.5*IQR of the score distribution (on the ~68k PBMC data this yields 4238 rare cells, with the top 0.25% of scores mapping exclusively to the megakaryocyte cluster). Because FiRE does not discriminate outliers from genuine minor cell types, downstream clustering (e.g. dropClust) of the selected rare cells is used to resolve true minor populations.

## Related tools

- **FiRE** (Core LSH-based rareness scorer: fits the Sketch model and assigns a rareness score to every individual expression profile in seconds (~31 s for ~68k profiles)) — https://github.com/princethewinner/FiRE
- **Python** (Primary environment for FiRE installation (./INSTALL --py), preprocessing (numpy, pandas, statsmodels), and scoring the expression matrix) — https://github.com/princethewinner/FiRE
- **dropClust** (Clusters the FiRE-detected rare cells after IQR thresholding to resolve them into minor clusters (e.g. R1-R13 dendritic clusters, R10 pars tuberalis sub-type))
- **RaceID** (Alternative rare-cell method offering binary predictions, used as a baseline comparison in F1-score evaluations on the Jurkat dilution simulations)
- **GiniClust** (Alternative rare-cell method offering binary predictions, used as a baseline comparison in F1-score evaluations on the Jurkat dilution simulations)

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489, verbose=0); model.fit(preprocessedData); score = np.array(model.score(preprocessedData))
```

## Evaluation signals

- fire_scores.csv contains exactly one finite numeric score row per cell in the input expression matrix.
- IQR thresholding (Q3 + 1.5*IQR) on the ~68k PBMC data yields the expected 4238 rare cells.
- The top 0.25% highest FiRE scores map exclusively to the smallest unambiguously annotated dropClust cluster (megakaryocytes, ~0.3% of cells) as in Fig. 2a / Supplementary Figure 1.
- On the 293T-Jurkat dilution simulations (0.5%, 2.5%, 5%), FiRE's F1 scores for the rare Jurkat population exceed those of LOF, RaceID and GiniClust at every dilution (Fig. 3a).
- Runtime invariant: scoring the ~68k PBMC dataset completes in seconds (~31 s reported), and the small jurkat_simulation_small demo finishes in ~4 s total.

## Limitations

- FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types; outlier cells get submerged into minor cell clusters when the rare cells are subsequently clustered with dropClust.
- FiRE's application to doublet detection is limited because doublet rates (1%-30%) vary widely and do not necessarily comply with the notion of cell rareness.
- The Python module with the --inplace option may fail uninstallation; the generated .so file must be removed manually.
- Loading the full demo data (data/jurkat_two_species_1580.txt.gz) requires a large amount of RAM; use the provided preprocessed file on constrained machines.

## Evidence

- [abstract] FiRE assigns a per-cell rareness score in seconds: "an algorithm that, in a matter of seconds, assigns a rareness score to every individual expression profile under study"
- [discussion] FiRE avoids clustering as an intermediate step and yields scores rather than binary calls: "While RaceID15 and GiniClust16 offer binary predictions, FiRE gives a rareness score to every individual expression profile."
- [readme] IQR thresholding defines rare vs non-rare calls: [evidence span withheld — anchor status: external_doc]
- [readme] Score interpretation: [evidence span withheld — anchor status: external_doc]
- [discussion] Limitation on outliers vs minor cell types: "FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types."
