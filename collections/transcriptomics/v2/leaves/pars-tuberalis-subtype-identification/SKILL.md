---
name: pars-tuberalis-subtype-identification
description: Use when a large scRNA-seq expression matrix (~tens of thousands of cells,
  e.g. ~20k Arc-ME mouse brain profiles from the Single Cell Portal) must be scanned
  for rare minor cell subpopulations that standard clustering would miss.
when_to_use_negative:
- Input is already a small pre-selected rare-cell set where density-based rareness
  scoring adds no value
- The goal is doublet detection — doublet rates (1–30%) do not necessarily comply
  with the notion of cell rareness, limiting FiRE's application there
- A comprehensive clustering of all cell types is required in a single pass; typical
  clustering techniques are time consuming and this skill targets minor subpopulations
  only
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: FiRE
  role: LSH-based density estimator assigning a rareness score to every individual
    expression profile; parameters L=100, M=50, H=1017881
  repo: github.com/princethewinner/FiRE
- name: dropClust
  role: Clusters the FiRE-detected rare cells into R1–R12 without special treatment
    of outliers
- name: Python
  role: Environment for installing FiRE, running ranger_preprocess, IQR thresholding,
    and marker characterization
  repo: https://github.com/princethewinner/FiRE
- name: R
  role: Alternative environment; FiRE is installable via install.packages('FiRE')
    with Rcpp and BH
  repo: https://github.com/princethewinner/FiRE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:9a30d2997d2562ab5eaf1cacc6dac7211710ebc7d97a8eb8a044dccdc5e38111
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  - R
  - dropClust
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pars-tuberalis-subtype-identification

## Summary

Identify novel rare-cell sub-types, such as the Hesx1+/Cyp2f2+ pars tuberalis sub-type, by scoring every cell for rareness with FiRE (LSH-based density estimation), thresholding scores via the IQR rule, and clustering the rare cells with dropClust to reveal minor cell types in the ~20k Arc-ME mouse brain scRNA-seq dataset.

## When to use

Use when a large scRNA-seq expression matrix (~tens of thousands of cells, e.g. ~20k Arc-ME mouse brain profiles from the Single Cell Portal) must be scanned for rare minor cell subpopulations that standard clustering would miss.

## When NOT to use

- Input is already a small pre-selected rare-cell set where density-based rareness scoring adds no value
- The goal is doublet detection — doublet rates (1–30%) do not necessarily comply with the notion of cell rareness, limiting FiRE's application there
- A comprehensive clustering of all cell types is required in a single pass; typical clustering techniques are time consuming and this skill targets minor subpopulations only

## Inputs

- ~20k Arc-ME mouse brain scRNA-seq expression matrix (cells x genes, from the Single Cell Portal, Campbell et al.)
- Gene name vector for ranger_preprocess variable-gene selection
- Optional: labels file marking known abundant vs rare cells for validation demos

## Outputs

- Per-cell FiRE rareness scores (higher = rarer)
- 727 rare cells selected by IQR thresholding (score >= Q3 + 1.5*IQR)
- dropClust cluster assignments R1–R12 for the rare cells
- arcme_rare_cell_clusters_markers.csv with rareness scores, cluster assignments, and marker expression (e.g. Hesx1, Cyp2f2, Alas2, Hba-a1, Hba-a2, Hbb-bt)

## How to apply

Load the ~20k Arc-ME mouse brain scRNA-seq expression matrix (Campbell et al., Single Cell Portal) as cells-by-genes and pre-process it with ranger_preprocess to keep the top 1000 variable genes (log2 transformed). Fit an FiRE model (e.g. L=100, M=50, H=1017881, seed=5489) on the preprocessed profiles and call model.score to assign a rareness score to every individual cell; higher scores indicate rarer cells. Threshold the scores using the IQR rule (score >= 75th percentile + 1.5*IQR) to retain the 727 rare cells. Cluster these FiRE-detected rare cells with dropClust, which administers no special treatment to outliers, into 12 clusters (R1–R12). Characterize clusters by marker expression, verifying R6 erythrocyte markers (Alas2, Hba-a1, Hba-a2, Hbb-bt) and the R10 Hesx1+/Cyp2f2+ pars tuberalis sub-type as in Fig. 6, and write rareness scores, cluster assignments, and marker expression to arcme_rare_cell_clusters_markers.csv.

## Related tools

- **FiRE** (LSH-based density estimator assigning a rareness score to every individual expression profile; parameters L=100, M=50, H=1017881) — github.com/princethewinner/FiRE
- **dropClust** (Clusters the FiRE-detected rare cells into R1–R12 without special treatment of outliers)
- **Python** (Environment for installing FiRE, running ranger_preprocess, IQR thresholding, and marker characterization) — https://github.com/princethewinner/FiRE
- **R** (Alternative environment; FiRE is installable via install.packages('FiRE') with Rcpp and BH) — https://github.com/princethewinner/FiRE

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489); model.fit(preprocessedData); score = np.array(model.score(preprocessedData))
```

## Evaluation signals

- IQR thresholding retains exactly the expected rare-cell count (727 cells from ~20k Arc-ME profiles)
- dropClust on the rare cells yields 12 clusters labeled R1–R12
- Cluster R6 expresses erythrocyte markers Alas2, Hba-a1, Hba-a2, Hbb-bt
- Cluster R10 co-expresses Cyp2f2 (parent pars tuber type 1C marker) with high Hesx1, matching the Fig. 6 novel pars tuberalis sub-type
- Rareness scores, cluster assignments, and marker expression round-trip correctly in arcme_rare_cell_clusters_markers.csv

## Limitations

- FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types; outlier cells may get submerged into minor cell clusters when clustering FiRE-detected rare cells with dropClust
- Doublet rate may vary between 1% and 30%, a considerably wide range that does not necessarily comply with the notion of cell rareness, limiting FiRE's application to doublet detection
- A truly rare cell type can only be found by profiling several thousands of cells; existing algorithms scale unbearably slowly or terminate at that sample size
- FiRE offers continuous rareness scores rather than binary predictions (unlike RaceID and GiniClust), so a thresholding rule such as IQR must be applied downstream

## Evidence

- [discussion] FiRE assigns rareness scores then dropClust clusters rare cells: "For clustering FiRE detected rare cells, in all our analyses, we adhered to dropClust24 that does not administer any special treatment to outliers."
- [abstract] Novel pars tuberalis sub-type recovered: "When applied to a large scRNA-seq dataset of mouse brain cells, FiRE recovered a novel sub-type of the pars tuberalis lineage."
- [readme] IQR thresholding of FiRE scores in README: "q3 = np.percentile(score, 75)
iqr = stats.iqr(score)
th = q3 + 1.5*iqr

indIqr = np.where(score >= th)[0]"
- [readme] FiRE model construction parameters: [evidence span withheld — anchor status: external_doc]
- [readme] Higher FiRE scores represent rare cells: "score : FiRE score of every cell : np.array[nCells]

Higher values of FiRE score represent rare cells."
- [discussion] FiRE limitation on outliers vs minor cell types: "FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types."
