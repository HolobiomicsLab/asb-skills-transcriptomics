---
name: villani-dendritic-subtype-classification
description: Use when FiRE-scored rare PBMC cells need dendritic sub-type resolution
  against Villani reference signatures, e.g. when rare-cell clusters must be assigned
  to DC1/DC3/DC4/DC6 identities.
when_to_use_negative:
- The rare-cell set has not been produced by the upstream FiRE rareness-scoring step
  on the ~68k PBMC dataset — this skill operates on FiRE-detected cells, not raw counts.
- The goal is de novo discovery of novel rare cell types rather than assigning cells
  to known Villani dendritic sub-type reference signatures.
- Only abundant (non-rare) PBMC populations are of interest; clustering the full ~68k
  profile set is a different analysis.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: FiRE
  role: Upstream rareness scorer that produced the 4238 rare PBMC cells (subset of
    ~68k profiles) that this skill clusters and characterizes
  repo: github.com/princethewinner/FiRE
- name: dropClust
  role: Clustering of the FiRE-detected rare cells into 13 clusters (R1–R13); chosen
    because it does not administer any special treatment to outliers
- name: Python
  role: Implementation environment for the differential expression analysis and writing
    dendritic_subtype_assignments.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:88096e68abc34ff53341bf4feb3655b4c781e7d20a877925ce23b6de8b82da53
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  - dropClust
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# villani-dendritic-subtype-classification

## Summary

Resolves Villani human blood dendritic cell sub-types (DC1, DC3, DC4, DC6) from FiRE-detected rare PBMC cells by clustering them with dropClust, isolating dendritic-exclusive clusters, and matching their differential genes to Villani reference signatures.

## When to use

Use when FiRE-scored rare PBMC cells need dendritic sub-type resolution against Villani reference signatures, e.g. when rare-cell clusters must be assigned to DC1/DC3/DC4/DC6 identities.

## When NOT to use

- The rare-cell set has not been produced by the upstream FiRE rareness-scoring step on the ~68k PBMC dataset — this skill operates on FiRE-detected cells, not raw counts.
- The goal is de novo discovery of novel rare cell types rather than assigning cells to known Villani dendritic sub-type reference signatures.
- Only abundant (non-rare) PBMC populations are of interest; clustering the full ~68k profile set is a different analysis.

## Inputs

- FiRE-detected rare cells (4238 cells) with rareness scores from the upstream FiRE step
- ~68k human PBMC single-cell expression profiles from which the rare cells were detected
- Villani dendritic cell sub-type reference gene signatures (six reported sub-types)

## Outputs

- Cluster-to-Villani-sub-type assignment table written to dendritic_subtype_assignments.csv
- Dendritic-exclusive cluster identities (R4, R8, R9, R13) with resolved sub-types DC1, DC3, DC4, DC6

## How to apply

Load the 4238 FiRE-detected rare cells together with their ~68k PBMC single-cell expression profiles produced by the upstream FiRE rareness-scoring step. Cluster only the rare-cell subset with dropClust, which applies no special treatment to outliers, yielding 13 clusters labeled R1–R13. Inspect cluster composition and retain only clusters populated exclusively by dendritic cells (R4, R8, R9, R13). Run differential gene expression analysis on these dendritic-exclusive clusters and overlay the differential genes with the dendritic sub-type signatures reported by Villani to assign each cluster to its best-matching sub-type following the Fig. 7 mapping. Write the cluster-to-sub-type assignment table to dendritic_subtype_assignments.csv.

## Related tools

- **FiRE** (Upstream rareness scorer that produced the 4238 rare PBMC cells (subset of ~68k profiles) that this skill clusters and characterizes) — github.com/princethewinner/FiRE
- **dropClust** (Clustering of the FiRE-detected rare cells into 13 clusters (R1–R13); chosen because it does not administer any special treatment to outliers)
- **Python** (Implementation environment for the differential expression analysis and writing dendritic_subtype_assignments.csv)

## Examples

```
model = FiRE.FiRE(L=100, M=50, H=1017881, seed=5489, verbose=0); model.fit(preprocessedData); score = np.array(model.score(preprocessedData))
```

## Evaluation signals

- dropClust partitions the 4238 rare cells into exactly 13 clusters labeled R1–R13.
- Exactly four clusters (R4, R8, R9, R13) are populated exclusively by dendritic cells after composition inspection.
- Four sub-types (DC1, DC3, DC4, DC6) out of the six Villani-reported sub-types are confidently resolved via the Fig. 7 mapping.
- The cluster-to-sub-type table dendritic_subtype_assignments.csv exists and contains a mapping for each dendritic-exclusive cluster.
- Differential genes of each dendritic-exclusive cluster overlap the corresponding Villani sub-type signature genes.

## Limitations

- FiRE does not discriminate between an outlier and cells representing minor cell types, so outlier cells may get submerged into minor cell clusters when clustering FiRE-detected rare cells with dropClust.
- Only 4 of the 6 Villani dendritic sub-types can be confidently resolved; DC2 and the remaining sub-types are not resolved by this workflow.
- Confident sub-type resolution depends on overlap between FiRE-cluster differential genes and published Villani signatures — novel or divergent sub-types will not map.

## Evidence

- [intro] dendritic clusters resolved to Villani sub-types: "We also demonstrated the efficacy of FiRE in delineating human blood dendritic cell sub-types using ~68 k single-cell expression profiles of human blood cells."
- [discussion] dropClust clustering choice: "For clustering FiRE detected rare cells, in all our analyses, we adhered to dropClust24 that does not administer any special treatment to outliers."
- [discussion] outlier submersion caveat: "As a result, outlier cells, if any, get submerged into the minor cell clusters."
- [discussion] FiRE runtime on PBMC data: "FiRE took ~31 s to analyze a scRNA-seq dataset containing ~68 k expression profiles."
- [readme] FiRE scoring mechanism: [evidence span withheld — anchor status: external_doc]
