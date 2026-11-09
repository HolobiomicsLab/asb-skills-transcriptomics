---
name: rare-cell-clustering-with-dropclust
description: Use when you already have per-cell rareness scores (FiRE) and IQR-thresholded
  rare cells (e.g. 727 cells from ~20k Arc-ME profiles) and need to partition them
  into minor sub-clusters for marker-based annotation.
when_to_use_negative:
- You need to detect rare cells from the full dataset first — that is the FiRE scoring/thresholding
  step, not clustering; FiRE scores must exist before clustering the subset.
- Your goal is to identify or remove technical outliers/doublets — dropClust gives
  outliers no special treatment and they get submerged into minor cell clusters, and
  FiRE does not discriminate between an outlier and cells representing minor cell
  types.
- The cell population you care about is abundant, not rare — a typical clustering
  technique cannot comprehensively chart minor cell types in one pass, and dropClust
  on the full ~20k profiles would not isolate the minor subpopulations this skill
  targets.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FiRE
  role: Assigns a rareness score to every expression profile; scores are IQR-thresholded
    to select the rare-cell subset (727 cells) that is then clustered
  repo: github.com/princethewinner/FiRE
- name: dropClust
  role: Clusters the FiRE-detected rare cells into R1-R12 subpopulations without special
    outlier treatment
- name: Python
  role: Environment for running FiRE scoring, ranger_preprocess, IQR thresholding,
    and downstream marker analysis
  repo: github.com/princethewinner/FiRE
- name: R
  role: Alternative environment for FiRE installation and rare-cell workflow
  repo: github.com/princethewinner/FiRE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:6489aa5834a7e80df282376c87cc9f7cb6b00b39de80eefe70b88d9210b78015
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  - R
  - dropClust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# rare-cell-clustering-with-dropClust

## Summary

Cluster the small set of FiRE-detected rare cells (e.g. the 727 rare cells from the ~20k Arc-ME mouse brain scRNA-seq dataset) into minor subpopulations using dropClust, which applies no special handling of outliers, to resolve novel minor cell types such as the Hesx1+/Cyp2f2+ pars tuberalis sub-type (cluster R10).

## When to use

Use when you already have per-cell rareness scores (FiRE) and IQR-thresholded rare cells (e.g. 727 cells from ~20k Arc-ME profiles) and need to partition them into minor sub-clusters for marker-based annotation.

## When NOT to use

- You need to detect rare cells from the full dataset first — that is the FiRE scoring/thresholding step, not clustering; FiRE scores must exist before clustering the subset.
- Your goal is to identify or remove technical outliers/doublets — dropClust gives outliers no special treatment and they get submerged into minor cell clusters, and FiRE does not discriminate between an outlier and cells representing minor cell types.
- The cell population you care about is abundant, not rare — a typical clustering technique cannot comprehensively chart minor cell types in one pass, and dropClust on the full ~20k profiles would not isolate the minor subpopulations this skill targets.

## Inputs

- Full scRNA-seq expression matrix (~20k Arc-ME mouse brain cells, samples x genes) from the Single Cell Portal (Campbell et al.)
- Per-cell FiRE rareness scores (np.array[nCells])
- IQR-thresholded subset of rare cells (e.g. 727 cells; th = q3 + 1.5*IQR on FiRE scores)
- Log2-transformed preprocessed data on ~1000 variable genes (ranger_preprocess output, preprocessedData and selGenes)

## Outputs

- Rare-cell cluster assignments R1-R12 for the 727 FiRE-detected rare cells
- Cluster marker-expression characterizations (e.g. R6 erythrocyte markers Alas2/Hba-a1/Hba-a2/Hbb-bt; R10 Hesx1+/Cyp2f2+ pars tuberalis sub-type)
- arcme_rare_cell_clusters_markers.csv with per-cell rareness scores, cluster assignments, and marker expression

## How to apply

First run FiRE (LSH-based density estimator, e.g. L=100, M=50, H=1017881, seed=5489) on the ~20k Arc-ME expression matrix from the Single Cell Portal (Campbell et al.) to score every cell, then threshold scores with the IQR rule (th = q3 + 1.5*IQR) to retain the 727 rare cells. Subset the preprocessed expression matrix (log2-transformed, top ~1000 variable genes from ranger_preprocess) to only these rare cells and cluster them with dropClust, yielding 12 clusters labeled R1-R12; dropClust is chosen precisely because it administers no special treatment to outliers, so outlier cells submerge into minor clusters rather than dominating them. Annotate clusters by marker expression, verifying known signatures (R6 erythrocyte markers Alas2, Hba-a1, Hba-a2, Hbb-bt) and looking for novel combinations such as the R10 Hesx1+/Cyp2f2+ pars tuberalis sub-type, cross-referenced with Allen Mouse Brain Atlas in situ hybridization data. Write per-cell rareness scores, cluster assignments, and marker expression to arcme_rare_cell_clusters_markers.csv.

## Related tools

- **FiRE** (Assigns a rareness score to every expression profile; scores are IQR-thresholded to select the rare-cell subset (727 cells) that is then clustered) — github.com/princethewinner/FiRE
- **dropClust** (Clusters the FiRE-detected rare cells into R1-R12 subpopulations without special outlier treatment)
- **Python** (Environment for running FiRE scoring, ranger_preprocess, IQR thresholding, and downstream marker analysis) — github.com/princethewinner/FiRE
- **R** (Alternative environment for FiRE installation and rare-cell workflow) — github.com/princethewinner/FiRE

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489, verbose=0); model.fit(preprocessedData); score = np.array(model.score(preprocessedData))
```

## Evaluation signals

- The rare-cell subset size matches the IQR rule expectation (727 rare cells from ~20k Arc-ME profiles; threshold th = q3 + 1.5*IQR on FiRE scores).
- Clustering the 727 cells yields the expected R1-R12 cluster structure (12 clusters) reproducible with dropClust.
- Marker sanity checks pass: R6 clusters express erythrocyte markers Alas2, Hba-a1, Hba-a2, Hbb-bt.
- A novel lineage cluster is recoverable: R10 co-expresses Cyp2f2 (marker of the parent pars tuber type 1C cell type) and high Hesx1, consistent with Allen Mouse Brain Atlas in situ hybridization.
- Output file arcme_rare_cell_clusters_markers.csv contains per-cell rareness scores, cluster assignments, and marker expression for all 727 cells.

## Limitations

- FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types; when clustering FiRE-detected rare cells with dropClust, outlier cells, if any, get submerged into the minor cell clusters.
- Dropout/limitation on doublets: doublet rate may vary between 1% and 30%, a considerably wide range that does not necessarily comply with the notion of cell rareness, limiting FiRE's application to doublet detection.
- Scalable rare-cell detection context: existing algorithms (RaceID, GiniClust) scale unbearably slowly or terminate as sample size grows to the order of tens of thousands, so they are unsuitable substitutes on inputs of this size.

## Evidence

- [discussion] For clustering FiRE detected rare cells, in all our analyses, we adhered to dropClust24 that does not administer any special treatment to outliers.: "For clustering FiRE detected rare cells, in all our analyses, we adhered to dropClust24 that does not administer any special treatment to outliers."
- [discussion] FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types.: "FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types."
- [abstract] When applied to a large scRNA-seq dataset of mouse brain cells, FiRE recovered a novel sub-type of the pars tuberalis lineage.: "When applied to a large scRNA-seq dataset of mouse brain cells, FiRE recovered a novel sub-type of the pars tuberalis lineage."
- [readme] IQR thresholding code from README: "q3 = np.percentile(score, 75)
iqr = stats.iqr(score)
th = q3 + 1.5*iqr"
- [readme] Higher FiRE scores mark rare cells: [evidence span withheld — anchor status: external_doc]
