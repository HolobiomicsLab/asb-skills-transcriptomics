---
name: marker-gene-cell-type-annotation
description: Use when a minor/rare cell cluster has been detected in a large scRNA-seq
  dataset (e.g. by GapClust on the 68k PBMC matrix) and must be annotated to a cell
  type via marker-gene expression (CD14, CD34) and benchmarked against GiniClust/FiRE.
when_to_use_negative:
- Annotating abundant, major cell types comprehensively — GapClust and this skill
  target rare/minor clusters hidden in voluminous data, not mainstream populations
  already characterized with Seurat or Scater
- The input is already a labeled cell-type annotation table or feature table rather
  than a raw/processed scRNA-seq expression matrix
- Datasets too small or datasets where GiniClust/FiRE-scale comparison is the goal
  (e.g. FiRE flags outliers, not minor cell types, so cross-checking on small data
  is not informative)
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GapClust
  role: Detects the rare candidate clusters (skewness_k > 2) whose marker expression
    is then annotated; rerun with detected clusters removed to find hidden clusters
  repo: github.com/fabotao/GapClust
- name: Seurat
  role: Toolkit for handling the expression matrix and marker-gene-based cell-type
    annotation (GapClust requires Seurat >= 3.1.0)
- name: GiniClust
  role: Competing rare-cell method whose outputs are cross-checked to confirm R1 (3-cell
    CD14+ monocyte cluster) is missed by it
- name: FiRE
  role: Competing rare-cell method cross-checked to confirm the 3-cell R1 cluster
    is missed by it (FiRE does not discriminate outliers from minor cell types)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:59f2fe163d027dd0ad5215f8a5960fdeea41b7ebd478491ae10dc99c693d1bd4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellRanger
  - FiRE
  - GapClust
  - GiniClust
  - Seurat
  - tauFisher
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# marker-gene-cell-type-annotation

## Summary

Assign cell-type identities to rare clusters detected from large scRNA-seq data (e.g. GapClust output on the 68k PBMC dataset) by checking expression of canonical marker genes such as CD14 and CD34, and cross-validate against competing rare-cell callers.

## When to use

Use when a minor/rare cell cluster has been detected in a large scRNA-seq dataset (e.g. by GapClust on the 68k PBMC matrix) and must be annotated to a cell type via marker-gene expression (CD14, CD34) and benchmarked against GiniClust/FiRE.

## When NOT to use

- Annotating abundant, major cell types comprehensively — GapClust and this skill target rare/minor clusters hidden in voluminous data, not mainstream populations already characterized with Seurat or Scater
- The input is already a labeled cell-type annotation table or feature table rather than a raw/processed scRNA-seq expression matrix
- Datasets too small or datasets where GiniClust/FiRE-scale comparison is the goal (e.g. FiRE flags outliers, not minor cell types, so cross-checking on small data is not informative)

## Inputs

- Full 68k PBMC single-cell expression matrix (count/normalized matrix compatible with Seurat)
- GapClust rare-cluster assignments with candidate ranking (skewness_k > 2)
- Competing rare-cell-detection outputs from GiniClust and FiRE for cross-checking
- Canonical marker-gene panel (e.g. CD14, CD34, erythrocyte markers)

## Outputs

- Cell-type labels for rare clusters (e.g. 3-cell CD14+ monocyte cluster R1; 185-cell CD34+ megakaryocyte cluster R2)
- Cluster sizes and marker-gene expression profiles per rare cluster
- rare_cells_68k_pbmc.csv with cluster assignments, sizes, and marker profiles

## How to apply

Run GapClust (github.com/fabotao/GapClust) on the full 68k PBMC expression matrix with its default candidate-selection criterion (skewness_k > 2) to rank candidate rare clusters. Annotate the top-ranked minor clusters by inspecting marker expression: R1 is confirmed as CD14+ monocytes (3 cells) and R2 as CD34+ megakaryocytes (185 cells). Cross-check R1 against GiniClust and FiRE outputs to confirm this 3-cell cluster is missed by both competing methods, and compare erythrocyte-marker expression of the 185 R2 cells against the remaining CD34+ cells to validate the Fig. 6a-d marker pattern. Validation succeeds when detected cluster sizes (3 and 185 cells) and marker signatures (CD14+ for R1; CD34+/megakaryocyte plus erythrocyte-marker pattern for R2) reproduce the reported results. Seurat (>= 3.1.0 per the GapClust README) is the standard toolkit for handling the expression matrix and marker-based annotation.

## Related tools

- **GapClust** (Detects the rare candidate clusters (skewness_k > 2) whose marker expression is then annotated; rerun with detected clusters removed to find hidden clusters) — github.com/fabotao/GapClust
- **Seurat** (Toolkit for handling the expression matrix and marker-gene-based cell-type annotation (GapClust requires Seurat >= 3.1.0))
- **GiniClust** (Competing rare-cell method whose outputs are cross-checked to confirm R1 (3-cell CD14+ monocyte cluster) is missed by it)
- **FiRE** (Competing rare-cell method cross-checked to confirm the 3-cell R1 cluster is missed by it (FiRE does not discriminate outliers from minor cell types))

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- Detected rare cluster sizes match the reported values exactly: 3 cells for R1 and 185 cells for R2 in the 68k PBMC dataset
- R1 cells express CD14+ monocyte markers and R2 cells express CD34+/megakaryocyte markers with the reported erythrocyte-marker pattern (Fig. 6a-d)
- The 3-cell R1 cluster is absent from both GiniClust and FiRE outputs, confirming GapClust's uniquely higher resolution
- Candidate rare clusters were selected with the default GapClust criterion skewness_k > 2
- Results are persisted as rare_cells_68k_pbmc.csv containing cluster assignments, sizes, and marker profiles

## Limitations

- GiniClust was reported to fail in processing very large datasets, limiting its use as a cross-check on the full 68k PBMC scale
- FiRE does not discriminate outliers from minor cell types, so its calls cannot serve as cell-type annotations directly
- No version numbers for compared tools (GiniClust, RaceID, CellSIUS, FiRE, EDGE) and no GapClust run parameters (e.g. K, skewness threshold) beyond skewness_k > 2 are reported
- Results for several real datasets are deferred to unlinked supplementary figures (Supplementary Figs. 14-22, Supplementary Table 5), limiting reproducibility of those validations

## Evidence

- [abstract] GapClust identifies tuft cells in an intestine dataset and a previously unrecognised subtype of monocyte in the 68k PBMC dataset: "GapClust identifies the tuft cells and a previously unrecognised subtype of monocyte, respectively"
- [discussion] Tuft cells validated by known markers: "Tuft cells are identified in the intestinal dataset, validated by existing maker genes including Dclk1 and Trpm5"
- [readme] GapClust learns rare cells without searching for informative genes: "GapClust does not
struggle to search for rare cell informative genes like most of the competitors, but learns the cluster size as well as rare cells"
- [intro] Limitations of competing rare-cell methods: [evidence span withheld — anchor status: not_found]
- [discussion] Default candidate-selection criterion: "Candidate k can be identified if the skewness<k> > 2."
