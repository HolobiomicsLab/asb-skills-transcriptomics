---
name: differential-expression-gene-identification
description: Use when you have scRNA-seq count matrices with known rare-cell labels
  (e.g., artificial splatter data with group.prob = c(0.99, 0.01) or the packaged
  1%Jurkat.h5) and need a validated set of DE genes and non-DE background genes for
  robustness/sensitivity tests.
when_to_use_negative:
- The goal is primary rare-cell discovery on unlabeled data with no known rare population
  to define a DE gene set against
- You only need scCAD's default end-to-end rare-cell calling without quantifying sensitivity
  to the number of DE genes available
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scCAD
  role: Target method whose rare-cell F1 is measured as a function of the number of
    substituted DE genes (rare_h=0.01, merge via median nearest-neighbor center distance,
    I=0.7)
  repo: https://github.com/xuyp-csu/scCAD
- name: FiRE
  role: Comparison method scored by F1 across DE-gene substitution levels
  repo: https://github.com/princethewinner/FiRE
- name: GapClust
  role: Comparison method scored by F1 across DE-gene substitution levels
  repo: https://github.com/fabotao/GapClust
- name: GiniClust3
  role: Comparison method scored by F1 across DE-gene substitution levels (stabilizes
    near F1 ~ 0.6 on Jurkat)
  repo: https://github.com/rdong08/GiniClust3
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:de91534071e61c9172019b03b96b270364c8dca65b94ae778baa1bec3cc3a12b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xuyp-csu/scCAD
  repo_url: https://github.com/xuyp-csu/scCAD
  tools:
  - FiRE
  - GapClust
  - GiniClust
  - GiniClust3
  - scCAD
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# differential-expression-gene-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies pre-confirmed differentially expressed (DE) genes distinguishing a rare cell population from the majority using a two-sided Wilcoxon rank sum test with FDR and fold-change cutoffs, then reserves or removes them for downstream sensitivity analyses (e.g., scCAD's DE-gene substitution robustness experiments).

## When to use

Use when you have scRNA-seq count matrices with known rare-cell labels (e.g., artificial splatter data with group.prob = c(0.99, 0.01) or the packaged 1%Jurkat.h5) and need a validated set of DE genes and non-DE background genes for robustness/sensitivity tests.

## When NOT to use

- The goal is primary rare-cell discovery on unlabeled data with no known rare population to define a DE gene set against
- You only need scCAD's default end-to-end rare-cell calling without quantifying sensitivity to the number of DE genes available

## Inputs

- scRNA-seq expression matrix (cells x genes) with rare-cell ground-truth labels, e.g., 1%Jurkat.h5 (1540 293T + 16 Jurkat cells) or an artificial splatter dataset (batchCells = 2500, nGenes = 5000, de.prob = c(0.4, 0.4), de.facLoc = 0.4, de.facScale = 0.8, seed = 2023)
- Cell-type/cluster label vector distinguishing rare from majority cells
- FDR cutoff (0.05) and absolute fold-change cutoff (1.5 artificial / 1 Jurkat)

## Outputs

- List of pre-identified reserved DE genes (e.g., 220 for artificial data, 108 for Jurkat)
- List of non-DE background genes (e.g., 3226 for artificial data, 30,479 for Jurkat)
- DE-gene-substituted expression matrices (one per count x 10 random repeats) and averaged F1 scores per method vs. DE-gene count
- Score table and F1-vs-DE-gene-count plot reproducing Supplementary Fig. 11

## How to apply

Run a two-sided Wilcoxon rank sum test between the rare population and the majority cells, retaining genes at FDR < 0.05 with absolute fold-change above a dataset-specific cutoff (1.5 for the artificial splatter dataset, relaxed to 1 for the Jurkat dataset). Split genes into a reserved DE set (220 DE / 3226 non-DE genes for the artificial data; 108 DE / 30,479 non-DE for Jurkat) and physically remove the DE genes from the expression matrices. For sensitivity testing, substitute back controlled numbers of reserved DE genes in place of randomly chosen non-DE genes, repeating each random substitution 10 times per tested DE-gene count, then score methods (scCAD with rare_h=0.01 and independence score threshold I=0.7, FiRE, GapClust, GiniClust3) by F1 on the rare cells. The rationale is to isolate how the availability of differential signal drives rare-cell detection performance across methods.

## Related tools

- **scCAD** (Target method whose rare-cell F1 is measured as a function of the number of substituted DE genes (rare_h=0.01, merge via median nearest-neighbor center distance, I=0.7)) — https://github.com/xuyp-csu/scCAD
- **FiRE** (Comparison method scored by F1 across DE-gene substitution levels) — https://github.com/princethewinner/FiRE
- **GapClust** (Comparison method scored by F1 across DE-gene substitution levels) — https://github.com/fabotao/GapClust
- **GiniClust3** (Comparison method scored by F1 across DE-gene substitution levels (stabilizes near F1 ~ 0.6 on Jurkat)) — https://github.com/rdong08/GiniClust3

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- Wilcoxon test with FDR < 0.05 and |fold-change| >= 1.5 (artificial) / 1 (Jurkat) yields the expected counts: 220 reserved DE and 3226 non-DE genes (artificial), 108 reserved DE and 30,479 non-DE genes (Jurkat)
- Each tested DE-gene count is averaged over 10 random substitution repetitions before computing mean F1
- Average-F1 curves qualitatively reproduce Supplementary Fig. 11: all methods improve with more DE genes and scCAD reaches higher F1 than FiRE, GapClust, and GiniClust3 (GiniClust3 stabilizing near F1 ~ 0.6 on Jurkat)
- scCAD runs with documented defaults (rare_h=0.01, merge threshold via median nearest-neighbor center distance, independence score threshold I=0.7, seed=2023)

## Limitations

- All methods fail when very few DE genes are available; the skill measures degradation rather than rescuing performance in weak differential expression
- Fold-change cutoff is dataset-dependent (1.5 artificial vs 1 Jurkat) and must be recalibrated for new datasets
- Random DE-gene substitution introduces stochasticity requiring the 10-repetition averaging; results are seed-dependent (default seed=2023)

## Evidence

- [methods] Wilcoxon test parameters: [evidence span withheld — anchor status: not_found]
- [supplementary] Expected DE gene counts: [evidence span withheld — anchor status: not_found]
- [results] Robustness finding: [evidence span withheld — anchor status: not_found]
