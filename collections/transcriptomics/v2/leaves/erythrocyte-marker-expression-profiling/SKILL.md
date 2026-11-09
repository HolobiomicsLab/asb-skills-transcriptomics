---
name: erythrocyte-marker-expression-profiling
description: Use when a rare-cell analysis of the 68k PBMC dataset has produced a
  candidate CD34+/megakaryocyte cluster (e.g. ~185 cells) and its reported erythrocyte-marker
  signature must be verified against remaining CD34+ cells.
when_to_use_negative:
- When the input is a small dataset where exhaustive methods such as RaceID or GiniClust
  are feasible and the GapClust rare-cluster step is unnecessary
- When the target population is a major cell type — GapClust and this validation target
  rare minor clusters, not bulk-type annotation
- When clusters have already been biologically annotated and no marker-signature verification
  of the CD34+/megakaryocyte call is needed
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GapClust
  role: Detects the rare clusters (R1, R2) in the 68k PBMC data via its second-order
    derivative gap statistic; candidate k identified when skewness_k > 2
  repo: github.com/fabotao/GapClust
- name: GiniClust
  role: Competing rare-cell detector used to cross-check whether the 3-cell CD14+
    monocyte cluster R1 is missed
- name: FiRE
  role: Competing rare-cell detector used to cross-check that R1 is missed; matched
    GapClust on R2
- name: Seurat
  role: Major cell-type characterization toolkit used alongside the rare-cell analysis
    (prerequisite R package, >= 3.1.0)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:9e9277f8b25f24409a09e9d952cbc6b838e556b774fa7fd7aa56d4db4289b279
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - GapClust
  - GiniClust
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# erythrocyte-marker-expression-profiling

## Summary

Validation skill for rare-cell discoveries in the 68k PBMC dataset: profile erythrocyte-marker expression in the GapClust-detected 185-cell CD34+ megakaryocyte cluster (R2) versus remaining CD34+ cells, replicating the marker pattern reported in Fig. 6a-d of the GapClust paper.

## When to use

Use when a rare-cell analysis of the 68k PBMC dataset has produced a candidate CD34+/megakaryocyte cluster (e.g. ~185 cells) and its reported erythrocyte-marker signature must be verified against remaining CD34+ cells.

## When NOT to use

- When the input is a small dataset where exhaustive methods such as RaceID or GiniClust are feasible and the GapClust rare-cluster step is unnecessary
- When the target population is a major cell type — GapClust and this validation target rare minor clusters, not bulk-type annotation
- When clusters have already been biologically annotated and no marker-signature verification of the CD34+/megakaryocyte call is needed

## Inputs

- Full 68k PBMC single-cell expression matrix (counts) from the 10x Genomics PBMC dataset
- GapClust cluster assignments for rare clusters R1 (3 cells) and R2 (185 cells)
- Marker-gene lists: CD14+ monocyte markers, CD34+/megakaryocyte markers, erythrocyte markers
- GiniClust and FiRE rare-cell outputs for cross-comparison

## Outputs

- rare_cells_68k_pbmc.csv with cluster assignments, cluster sizes (3 and 185 cells), and per-cluster marker profiles
- Erythrocyte-marker expression comparison of the 185 R2 cells vs. remaining CD34+ cells (Fig. 6a-d analogue)
- Confirmation table showing R1 (CD14+ monocytes) is missed by GiniClust and FiRE

## How to apply

After GapClust (installed from github.com/fabotao/GapClust, requiring Seurat >= 3.1.0) identifies rare clusters using its candidate-selection criterion (skewness_k > 2), annotate the top-ranked minor clusters: confirm R1 (3 CD14+ monocytes) and R2 (185 CD34+ megakaryocytes). Then, per the article's validation logic, compare the erythrocyte-marker expression profile of the 185 R2 cells against the remaining CD34+ cells to reproduce the Fig. 6a-d pattern; marker validation follows the paper's practice of confirming populations with known marker genes (e.g. Dclk1/Trpm5 for tuft cells in the intestine dataset). Cross-check R1 against GiniClust and FiRE outputs, since the 3-cell CD14+ monocyte cluster was detected only by GapClust. Save cluster assignments, sizes, and marker profiles to rare_cells_68k_pbmc.csv. This mirrors the paper's rationale that rare-cell calls should be validated with existing marker genes rather than by clustering alone.

## Related tools

- **GapClust** (Detects the rare clusters (R1, R2) in the 68k PBMC data via its second-order derivative gap statistic; candidate k identified when skewness_k > 2) — github.com/fabotao/GapClust
- **GiniClust** (Competing rare-cell detector used to cross-check whether the 3-cell CD14+ monocyte cluster R1 is missed)
- **FiRE** (Competing rare-cell detector used to cross-check that R1 is missed; matched GapClust on R2)
- **Seurat** (Major cell-type characterization toolkit used alongside the rare-cell analysis (prerequisite R package, >= 3.1.0))

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- Rare-cluster sizes reproduce the reported values: R1 = 3 cells, R2 = 185 cells
- R1 cells express CD14+ monocyte markers; R2 cells express CD34+/megakaryocyte markers
- Erythrocyte-marker expression in the 185 R2 cells matches the pattern of Fig. 6a-d when compared to the remaining CD34+ cells
- R1 is not present in GiniClust or FiRE outputs, confirming the 'detected only by GapClust' claim
- Output file rare_cells_68k_pbmc.csv exists with columns for cluster assignment, size, and marker profile

## Limitations

- No parameter settings for GapClust runs (e.g. K, skewness threshold) are reported in all sections of the paper, so exact reproducibility of thresholds may require the default skewness_k > 2
- Results for many real datasets are deferred to unlinked supplementary figures (Supplementary Figs. 14-22, Supplementary Table 5), limiting the available validation detail
- FiRE does not discriminate outliers from minor cell types, so cross-method agreement is imperfect as a validation signal

## Evidence

- [abstract] When applying our algorithm to an intestine and 68 k PBMC datasets, GapClust identifies the tuft cells and a previously unrecognised subtype of monocyte: "When applying our algorithm to an intestine and 68 k PBMC datasets, GapClust identifies the tuft cells and a previously unrecognised subtype of monocyte, respectively"
- [discussion] Tuft cells validated with marker genes: "Tuft cells are identified in the intestinal dataset, validated by existing maker genes including Dclk1 and Trpm5"
- [discussion] Skewness criterion for candidate k: "Candidate k can be identified if the skewness<k> > 2."
- [readme] README introduction to the gap statistic: "GapClust takes advantage of the gap between minor cluster and neighbouring abundant cluster to let rare cells within minor cluster stand out through delicately designed statistics."
- [readme] README installation command: "devtools::install_github("fabotao/GapClust")"
