---
name: cluster-marker-characterization
description: Use when a set of candidate rare cells (e.g., 727 FiRE-scored cells from
  ~20k Arc-ME mouse brain profiles) must be partitioned into subpopulations and assigned
  cell-type identities via marker genes.
when_to_use_negative:
- The input is a full heterogeneous scRNA-seq dataset needing global clustering of
  all major cell types — FiRE/rare-cell detection plus dropClust on the rare subset
  targets minor subpopulations, not comprehensive charting of a complex tissue in
  a single pass
- The goal is to distinguish technical outliers from genuine rare cell types — FiRE
  does not discriminate between an outlier and cells representing minor cell types,
  and outlier cells may get submerged into minor cell clusters when clustered with
  dropClust
- Binary rare-cell calls suffice and no sub-type resolution is needed — RaceID and
  GiniClust already offer binary predictions; this skill's added value is marker-based
  sub-type characterization of the rare fraction
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: dropClust
  role: Clusters the FiRE-detected rare cells into subpopulations (R1-R12); chosen
    because it does not administer any special treatment to outliers
- name: FiRE
  role: Upstream step that assigns a rareness score to every expression profile and
    defines the rare-cell subset passed to clustering
  repo: github.com/princethewinner/FiRE
- name: Python
  role: Environment for running FiRE (all manuscript results generated with the python
    module) and writing per-cell score/cluster/marker tables
  repo: github.com/princethewinner/FiRE
- name: R
  role: Alternative environment (FiRE R package via install.packages('FiRE')) for
    scoring and downstream dropClust clustering
  repo: github.com/princethewinner/FiRE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:233505ce320dbfa75fabfa9c6bc12d956f3f1d39378a117becbb2de4c0583624
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  - R
  - dropClust
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cluster-marker-characterization

## Summary

Clusters a set of FiRE-detected rare cells with dropClust into labeled subpopulations (e.g., R1-R12) and characterizes each cluster by marker gene expression to identify novel minor cell types, such as the Hesx1+/Cyp2f2+ pars tuberalis sub-type (cluster R10).

## When to use

Use when a set of candidate rare cells (e.g., 727 FiRE-scored cells from ~20k Arc-ME mouse brain profiles) must be partitioned into subpopulations and assigned cell-type identities via marker genes.

## When NOT to use

- The input is a full heterogeneous scRNA-seq dataset needing global clustering of all major cell types — FiRE/rare-cell detection plus dropClust on the rare subset targets minor subpopulations, not comprehensive charting of a complex tissue in a single pass
- The goal is to distinguish technical outliers from genuine rare cell types — FiRE does not discriminate between an outlier and cells representing minor cell types, and outlier cells may get submerged into minor cell clusters when clustered with dropClust
- Binary rare-cell calls suffice and no sub-type resolution is needed — RaceID and GiniClust already offer binary predictions; this skill's added value is marker-based sub-type characterization of the rare fraction

## Inputs

- Expression matrix (cells x genes) subset to FiRE-detected rare cells (e.g., 727 cells from the ~20k Arc-ME mouse brain scRNA-seq data from the Single Cell Portal, Campbell et al.)
- Per-cell FiRE rareness scores
- List of candidate marker genes (e.g., Alas2, Hba-a1, Hba-a2, Hbb-bt, Hesx1, Cyp2f2)

## Outputs

- Cluster assignments for each rare cell (clusters R1-R12)
- Per-cluster marker expression profiles
- Identified novel sub-types (e.g., R10: Hesx1+/Cyp2f2+ pars tuberalis sub-type of pars tuber type 1C lineage)
- CSV table of per-cell rareness scores, cluster assignments, and marker expression (arcme_rare_cell_clusters_markers.csv)

## How to apply

Take the cell-by-gene expression matrix restricted to the rare cells identified by FiRE score thresholding (IQR rule: score >= Q3 + 1.5*IQR). Cluster these cells with dropClust, which does not administer any special treatment to outliers, yielding clusters labeled R1-R12 (12 clusters in the Arc-ME mouse brain case). Then characterize each cluster by marker expression: verify expected lineage markers (e.g., erythrocyte markers Alas2, Hba-a1, Hba-a2, Hbb-bt in cluster R6) and look for novel combinations — cluster R10 co-expresses Cyp2f2, the marker of its parent pars tuber type 1C lineage, with high Hesx1, defining a novel pars tuberalis sub-type. Cross-reference marker findings with in situ hybridization data (e.g., Allen Mouse Brain Atlas) to validate spatial/anatomical plausibility. Write per-cell rareness scores, cluster assignments, and marker expression to a table (e.g., arcme_rare_cell_clusters_markers.csv).

## Related tools

- **dropClust** (Clusters the FiRE-detected rare cells into subpopulations (R1-R12); chosen because it does not administer any special treatment to outliers)
- **FiRE** (Upstream step that assigns a rareness score to every expression profile and defines the rare-cell subset passed to clustering) — github.com/princethewinner/FiRE
- **Python** (Environment for running FiRE (all manuscript results generated with the python module) and writing per-cell score/cluster/marker tables) — github.com/princethewinner/FiRE
- **R** (Alternative environment (FiRE R package via install.packages('FiRE')) for scoring and downstream dropClust clustering) — github.com/princethewinner/FiRE

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489); model.fit(preprocessedData); score = np.array(model.score(preprocessedData)); indIqr = np.where(score >= np.percentile(score,75) + 1.5*stats.iqr(score))[0]
```

## Evaluation signals

- Number of FiRE-selected rare cells matches the IQR threshold count (727 of ~20k Arc-ME cells) and dropClust partitions them into 12 clusters R1-R12
- Cluster R6 expresses the expected erythrocyte markers Alas2, Hba-a1, Hba-a2, and Hbb-bt, reproducing Fig. 6
- Cluster R10 co-expresses Cyp2f2 (parent pars tuber type 1C marker) and high Hesx1, matching the reported novel pars tuberalis sub-type
- Marker-based cell-type calls are corroborated by Allen Mouse Brain Atlas in situ hybridization data
- Output CSV contains per-cell rareness scores, cluster assignments, and marker expression with consistent cell identifiers across columns

## Limitations

- FiRE does not discriminate between an outlier and cells representing minor cell types; outlier cells, if any, get submerged into the minor cell clusters when clustering with dropClust
- Typical clustering techniques are time consuming and incapable of comprehensively charting minor cell types in a complex tissue in a single go — this skill characterizes only the pre-selected rare fraction
- FiRE's rareness notion does not necessarily comply with doublet-based artifacts, since doublet rate may vary between 1% and 30%

## Evidence

- [discussion] For clustering FiRE detected rare cells, in all our analyses, we adhered to dropClust24 that does not administer any special treatment to outliers.: "For clustering FiRE detected rare cells, in all our analyses, we adhered to dropClust24 that does not administer any special treatment to outliers."
- [abstract] When applied to a large scRNA-seq dataset of mouse brain cells, FiRE recovered a novel sub-type of the pars tuberalis lineage.: "When applied to a large scRNA-seq dataset of mouse brain cells, FiRE recovered a novel sub-type of the pars tuberalis lineage."
- [discussion] FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types.: "FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types."
- [intro] We cross-referenced our findings with in situ hybridization data obtained from the Allen Mouse Brain Atlas20.: "We cross-referenced our findings with in situ hybridization data obtained from the Allen Mouse Brain Atlas20."
- [readme] Higher values of FiRE score represent rare cells.: [evidence span withheld — anchor status: external_doc]
