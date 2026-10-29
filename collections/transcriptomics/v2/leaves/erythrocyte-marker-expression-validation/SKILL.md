---
name: erythrocyte-marker-expression-validation
description: Use when a rare-cell cluster from LSH-based rareness scoring (FiRE) plus
  dropClust clustering must be assigned a cell-type identity, and candidate marker
  genes (e.g. erythrocyte markers Alas2, Hba-a1, Hba-a2, Hbb-bt) are available to
  confirm the assignment.
when_to_use_negative:
- When the goal is de novo rare-cell discovery from raw data with no cluster identities
  yet — first run FiRE scoring and dropClust clustering; marker validation is a downstream
  confirmation step
- When testing whether a rare cell is a doublet or technical artifact — the article
  notes doublet rates vary between 1% and 30%, a range that does not necessarily comply
  with the notion of cell rareness
- When the candidate cluster's expected markers are not known in advance; this skill
  validates a hypothesized identity against specific named markers
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FiRE
  role: Assigns an LSH-based rareness score to every individual expression profile
    so rare cells can be thresholded (IQR rule) before marker validation
  repo: github.com/princethewinner/FiRE
- name: dropClust
  role: Clusters the FiRE-detected rare cells into groups (R1-R12) whose identities
    are then validated by marker expression; applies no special treatment to outliers
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:768e5238619875ecaffb156bf1ed864f68248807abb2fb88f4f4767ba6e6cb98
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - R
  - dropClust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Erythrocyte marker expression validation

## Summary

Validates that a rare-cell cluster identified by FiRE scoring and dropClust clustering of scRNA-seq data is a genuine erythrocyte population by confirming co-expression of the canonical erythrocyte markers Alas2, Hba-a1, Hba-a2, and Hbb-bt, as demonstrated for cluster R6 in the ~20k Arc-ME mouse brain dataset.

## When to use

Use when a rare-cell cluster from LSH-based rareness scoring (FiRE) plus dropClust clustering must be assigned a cell-type identity, and candidate marker genes (e.g. erythrocyte markers Alas2, Hba-a1, Hba-a2, Hbb-bt) are available to confirm the assignment.

## When NOT to use

- When the goal is de novo rare-cell discovery from raw data with no cluster identities yet — first run FiRE scoring and dropClust clustering; marker validation is a downstream confirmation step
- When testing whether a rare cell is a doublet or technical artifact — the article notes doublet rates vary between 1% and 30%, a range that does not necessarily comply with the notion of cell rareness
- When the candidate cluster's expected markers are not known in advance; this skill validates a hypothesized identity against specific named markers

## Inputs

- scRNA-seq expression matrix (rows = cells, columns = genes), e.g. ~20k Arc-ME mouse brain data from the Single Cell Portal (Campbell et al.)
- FiRE rareness scores for every cell (higher scores = rarer cells)
- dropClust cluster assignments for the IQR-thresholded rare cells (727 cells, clusters R1-R12)
- List of canonical erythrocyte marker genes: Alas2, Hba-a1, Hba-a2, Hbb-bt

## Outputs

- Per-cluster marker expression profiles for the rare-cell clusters (R1-R12)
- Binary validation decision per cluster: erythrocyte identity confirmed when the cluster expresses Alas2, Hba-a1, Hba-a2, and Hbb-bt
- Per-cell table of rareness scores, cluster assignments, and marker expression (e.g. arcme_rare_cell_clusters_markers.csv)

## How to apply

Run FiRE on the scRNA-seq expression matrix (cells as rows, top ~1000 variable genes from ranger_preprocess, log2-transformed) to assign a rareness score to every cell, then threshold scores with the IQR rule (threshold = q3 + 1.5*IQR) — in the Arc-ME mouse brain data this retained 727 rare cells. Cluster the retained cells with dropClust, which administers no special treatment to outliers, yielding clusters R1-R12. For each cluster, examine marker-gene expression to establish identity; a cluster is validated as erythrocyte-derived only if it co-expresses the erythrocyte markers Alas2, Hba-a1, Hba-a2, and Hbb-bt (as cluster R6 does in Fig. 6). Marker-based validation of multiple clusters (e.g. R10 as an Hesx1+/Cyp2f2+ pars tuberalis sub-type) cross-referenced with in situ hybridization data (Allen Mouse Brain Atlas) confirms that FiRE-detected rare populations correspond to real minor cell types rather than technical artifacts.

## Related tools

- **FiRE** (Assigns an LSH-based rareness score to every individual expression profile so rare cells can be thresholded (IQR rule) before marker validation) — github.com/princethewinner/FiRE
- **dropClust** (Clusters the FiRE-detected rare cells into groups (R1-R12) whose identities are then validated by marker expression; applies no special treatment to outliers)

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489, verbose=0); model.fit(preprocessedData); score = np.array(model.score(preprocessedData))
```

## Evaluation signals

- The rare-cell subset size matches the IQR-thresholding expectation (~727 cells from ~20k Arc-ME profiles)
- dropClust partitions the rare cells into 12 clusters labeled R1-R12
- The candidate erythrocyte cluster (R6) co-expresses all four markers Alas2, Hba-a1, Hba-a2, and Hbb-bt, not just a subset
- FiRE scores are higher for rare cells than abundant cells (README: higher FiRE score values represent rare cells)
- Marker-based identities (e.g. R6 erythrocyte, R10 Hesx1+/Cyp2f2+ pars tuberalis) are consistent with independent in situ hybridization evidence from the Allen Mouse Brain Atlas

## Limitations

- FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types
- Outlier cells may get submerged into minor cell clusters when clustering FiRE-detected rare cells with dropClust
- Doublet rate may vary between 1% and 30%, a wide range that does not necessarily comply with the notion of cell rareness, limiting FiRE's application to doublet detection
- Marker validation depends on a gene-by-gene check of a small rare-cell cluster; clusters without known marker combinations cannot be confirmed this way

## Evidence

- [discussion] Clustering rare cells then marker-checking: "For clustering FiRE detected rare cells, in all our analyses, we adhered to dropClust24 that does not administer any special treatment to outliers."
- [other] Erythrocyte marker set for R6: [evidence span withheld — anchor status: not_found]
- [readme] FiRE score interpretation: [evidence span withheld — anchor status: external_doc]
- [readme] IQR thresholding used to select rare cells: "q3 = np.percentile(score, 75)
iqr = stats.iqr(score)
th = q3 + 1.5*iqr"
- [abstract] Novel sub-type recovered in mouse brain data: "When applied to a large scRNA-seq dataset of mouse brain cells, FiRE recovered a novel sub-type of the pars tuberalis lineage."
