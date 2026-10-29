---
name: cluster-to-reference-signature-mapping
description: Use when you have clusters of rare cells detected de novo (e.g. from
  FiRE + dropClust) and a published reference signature (e.g. Villani dendritic sub-types)
  to which you must assign identities via differential gene overlap.
when_to_use_negative:
- Input cells have not been clustered (the skill requires pre-existing cluster labels
  such as dropClust R1–R13)
- No published reference signature exists for the candidate cell sub-types, making
  differential-gene overlay impossible
- Goal is to detect rare cells from scratch rather than annotate existing clusters
  — use FiRE rareness scoring first
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: FiRE
  role: Upstream detector that assigns a rareness score to every expression profile,
    producing the rare cells that are subsequently clustered and mapped to reference
    sub-types
  repo: github.com/princethewinner/FiRE
- name: dropClust
  role: Clusters the FiRE-detected rare cells (no special treatment to outliers) into
    R1–R13 so dendritic-exclusive clusters can be identified
- name: Python
  role: Environment for running FiRE and downstream differential expression and assignment-table
    writing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:fc5de0f9cdc1224292460b289885b44ad2ebe11f7720eeef222a075100f7e4c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  - dropClust
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cluster-to-reference-signature-mapping

## Summary

Assigns de novo clusters of rare cells to published cell-type sub-type identities by differential expression overlap with reference signatures, e.g. resolving FiRE-detected dendritic clusters (R4, R8, R9, R13) into Villani sub-types DC1, DC3, DC4, DC6.

## When to use

Use when you have clusters of rare cells detected de novo (e.g. from FiRE + dropClust) and a published reference signature (e.g. Villani dendritic sub-types) to which you must assign identities via differential gene overlap.

## When NOT to use

- Input cells have not been clustered (the skill requires pre-existing cluster labels such as dropClust R1–R13)
- No published reference signature exists for the candidate cell sub-types, making differential-gene overlay impossible
- Goal is to detect rare cells from scratch rather than annotate existing clusters — use FiRE rareness scoring first

## Inputs

- FiRE-detected rare cell expression profiles (e.g. 4238 rare cells from ~68k human PBMC single-cell expression profiles)
- dropClust cluster labels (R1–R13) for the rare cells
- Published reference cell sub-type differential gene signatures (Villani dendritic cell sub-types)

## Outputs

- Identified dendritic-exclusive clusters (R4, R8, R9, R13)
- Cluster-to-reference-sub-type assignment table (dendritic_subtype_assignments.csv) mapping clusters to DC1/DC3/DC4/DC6
- Differential gene sets per dendritic-exclusive cluster

## How to apply

First cluster the FiRE-detected rare cells with dropClust, which applies no special treatment to outliers, yielding clusters labeled R1–R13. Identify the clusters populated exclusively by dendritic cells (R4, R8, R9 and R13) by inspecting cluster composition. Run differential gene expression analysis between these dendritic-exclusive clusters and the Villani dendritic sub-type reference signatures, then overlay the differential genes of each FiRE-detected cluster with those reported by Villani. Assign each cluster to its best-matching Villani sub-type following the Fig. 7 mapping, accepting only confident resolutions (four of six reported sub-types: DC1, DC3, DC4, DC6) and writing the cluster-to-sub-type table to a CSV (e.g. dendritic_subtype_assignments.csv).

## Related tools

- **FiRE** (Upstream detector that assigns a rareness score to every expression profile, producing the rare cells that are subsequently clustered and mapped to reference sub-types) — github.com/princethewinner/FiRE
- **dropClust** (Clusters the FiRE-detected rare cells (no special treatment to outliers) into R1–R13 so dendritic-exclusive clusters can be identified)
- **Python** (Environment for running FiRE and downstream differential expression and assignment-table writing)

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489, verbose=0); model.fit(preprocessedData); score = np.array(model.score(preprocessedData))
```

## Evaluation signals

- Cluster composition inspection confirms R4, R8, R9 and R13 are populated exclusively by dendritic cells
- Overlay of cluster differential genes with Villani signatures yields confident assignments for exactly 4 of the 6 reported sub-types (DC1, DC3, DC4, DC6)
- Each dendritic-exclusive cluster maps to exactly one Villani sub-type in the Fig. 7 mapping
- Output table dendritic_subtype_assignments.csv contains a complete cluster-to-sub-type row set for R4, R8, R9, R13
- Unresolved sub-types are not forced: only confidently resolvable sub-types are assigned

## Limitations

- Only 4 of the 6 Villani-reported dendritic sub-types could be confidently resolved; finer sub-types may not be distinguishable at this rare-cell resolution
- Outlier cells may get submerged into minor cell clusters when clustering FiRE-detected rare cells with dropClust, since dropClust administers no special treatment to outliers
- FiRE in principle does not discriminate between an outlier and cells representing minor cell types, so detected rare-cell sets may include non-biological outliers that contaminate cluster assignments

## Evidence

- [discussion] dropClust usage: "For clustering FiRE detected rare cells, in all our analyses, we adhered to dropClust24 that does not administer any special treatment to outliers."
- [intro] DC sub-type resolution goal: "We also demonstrated the efficacy of FiRE in delineating human blood dendritic cell sub-types using ~68 k single-cell expression profiles of human blood cells."
- [discussion] FiRE runtime on the 68k dataset: "FiRE took ~31 s to analyze a scRNA-seq dataset containing ~68 k expression profiles."
- [readme] IQR threshold selection of rare cells: "q3 = np.percentile(score, 75)
iqr = stats.iqr(score)
th = q3 + 1.5*iqr"
- [discussion] Outlier submersion caveat: "As a result, outlier cells, if any, get submerged into the minor cell clusters."
