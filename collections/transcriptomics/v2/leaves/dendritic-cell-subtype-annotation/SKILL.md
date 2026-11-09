---
name: dendritic-cell-subtype-annotation
description: Use when FiRE-scored ~68k human PBMC profiles have yielded a rare-cell
  subset (~4238 cells) and you need to resolve which Villani dendritic sub-types (DC1,
  DC3, DC4, DC6) the rare dendritic clusters correspond to.
when_to_use_negative:
- 'The goal is doublet detection: the article notes doublet rates vary between 1%
  and 30%, a range that does not necessarily comply with the notion of cell rareness,
  limiting FiRE''s application to doublet detection.'
- 'The rare-cell pool contains outliers that must be separated from minor cell types:
  FiRE does not discriminate between an outlier and cells representing minor cell
  types, and outliers get submerged into the minor cell clusters when clustering with
  dropClust.'
- 'You need all six Villani DC sub-types resolved: this skill confidently resolves
  only four (DC1, DC3, DC4, DC6) out of the six reported sub-types.'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FiRE
  role: Upstream rareness scorer that assigns a score to every individual expression
    profile of the ~68k PBMC dataset and flags the ~4238 rare cells subsequently clustered
    for dendritic sub-type resolution.
  repo: github.com/princethewinner/FiRE
- name: dropClust
  role: Clusters the FiRE-detected rare cells into 13 clusters (R1–R13); it does not
    administer any special treatment to outliers.
- name: Python
  role: Environment in which FiRE and the downstream differential-expression and assignment
    workflow are run (manuscript results were generated using the python module).
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:d10b0b61a33b777a72c26c37985dff2ad30c4ef1bc24ed61c52d717cfe00f622
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  - dropClust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# dendritic-cell-subtype-annotation

## Summary

Resolves dendritic cell (DC) sub-types within FiRE-detected rare PBMC cells by clustering the rare cells with dropClust, identifying the exclusively dendritic clusters, and matching their differential genes to Villani DC sub-type reference signatures.

## When to use

Use when FiRE-scored ~68k human PBMC profiles have yielded a rare-cell subset (~4238 cells) and you need to resolve which Villani dendritic sub-types (DC1, DC3, DC4, DC6) the rare dendritic clusters correspond to.

## When NOT to use

- The goal is doublet detection: the article notes doublet rates vary between 1% and 30%, a range that does not necessarily comply with the notion of cell rareness, limiting FiRE's application to doublet detection.
- The rare-cell pool contains outliers that must be separated from minor cell types: FiRE does not discriminate between an outlier and cells representing minor cell types, and outliers get submerged into the minor cell clusters when clustering with dropClust.
- You need all six Villani DC sub-types resolved: this skill confidently resolves only four (DC1, DC3, DC4, DC6) out of the six reported sub-types.

## Inputs

- FiRE-detected rare PBMC cells (~4238 cells) with their single-cell expression profiles
- Full ~68k human PBMC single-cell expression profile matrix from the upstream FiRE rareness-scoring step (cells scored rare at Q3 + 1.5*IQR threshold)
- Villani dendritic cell sub-type reference signatures (six reported sub-types)

## Outputs

- 13 rare-cell clusters labeled R1–R13 from dropClust
- Identification of dendritic-exclusive clusters (R4, R8, R9, R13)
- Differential gene lists per dendritic-exclusive cluster overlaid with Villani signatures
- Cluster-to-sub-type assignment table dendritic_subtype_assignments.csv mapping clusters to DC1, DC3, DC4, or DC6

## How to apply

Load the FiRE-detected rare cells and their parent ~68k PBMC single-cell expression profiles produced by the upstream FiRE rareness-scoring step (score threshold = Q3 + 1.5*IQR of FiRE scores). Cluster the rare cells with dropClust, which applies no special treatment to outliers, partitioning them into 13 clusters labeled R1–R13. Inspect cluster composition to find the clusters populated exclusively by dendritic cells (R4, R8, R9, R13). Run differential gene expression analysis on these dendritic-exclusive clusters and overlay their differential genes with the Villani dendritic sub-type reference signatures. Assign each cluster to its best-matching Villani sub-type following the Fig. 7 mapping, resolving four of the six reported sub-types (DC1, DC3, DC4, DC6), and write the assignments to dendritic_subtype_assignments.csv.

## Related tools

- **FiRE** (Upstream rareness scorer that assigns a score to every individual expression profile of the ~68k PBMC dataset and flags the ~4238 rare cells subsequently clustered for dendritic sub-type resolution.) — github.com/princethewinner/FiRE
- **dropClust** (Clusters the FiRE-detected rare cells into 13 clusters (R1–R13); it does not administer any special treatment to outliers.)
- **Python** (Environment in which FiRE and the downstream differential-expression and assignment workflow are run (manuscript results were generated using the python module).)

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489); model.fit(preprocessedData); score = np.array(model.score(preprocessedData))
```

## Evaluation signals

- dropClust partitions the FiRE-detected rare cells into exactly 13 clusters labeled R1–R13.
- Exactly four clusters (R4, R8, R9, R13) are populated exclusively by dendritic cells.
- Overlay of differential genes with Villani signatures confidently resolves four sub-types — DC1, DC3, DC4, DC6 — not all six.
- dendritic_subtype_assignments.csv contains one sub-type assignment (DC1/DC3/DC4/DC6) per dendritic-exclusive cluster consistent with the Fig. 7 mapping.
- Upstream rare-cell selection used the IQR criterion: score >= Q3 + 1.5*IQR of FiRE scores.

## Limitations

- FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types, so outlier cells, if any, get submerged into the minor cell clusters when the rare cells are clustered with dropClust.
- Only four of the six Villani-reported dendritic sub-types (DC1, DC3, DC4, DC6) could be confidently resolved; two sub-types remain unresolved.
- Typical clustering techniques are time consuming and incapable of comprehensively charting minor cell types in a complex tissue in a single pass, so sub-type resolution depends on the upstream rare-cell detection step.

## Evidence

- [intro] FiRE delineates human blood dendritic cell sub-types from ~68k profiles: "We also demonstrated the efficacy of FiRE in delineating human blood dendritic cell sub-types using ~68 k single-cell expression profiles of human blood cells."
- [discussion] dropClust used for clustering FiRE-detected rare cells: "For clustering FiRE detected rare cells, in all our analyses, we adhered to dropClust24 that does not administer any special treatment to outliers."
- [discussion] Outliers submerge into minor cell clusters: "As a result, outlier cells, if any, get submerged into the minor cell clusters."
- [readme] FiRE score semantics for selecting rare cells: [evidence span withheld — anchor status: external_doc]
- [results] Four of six Villani DC sub-types resolved (task finding): [evidence span withheld — anchor status: not_found]
