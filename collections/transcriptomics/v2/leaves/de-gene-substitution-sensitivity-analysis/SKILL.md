---
name: de-gene-substitution-sensitivity-analysis
description: Use when quantifying how the number of differentially expressed genes
  available to a rare-cell identification method affects its F1 score, e.g. testing
  scCAD's robustness in weak differential expression scenarios against FiRE, GapClust,
  and GiniClust3.
when_to_use_negative:
- When the goal is simply to call rare cells on a real dataset with its full complement
  of DE genes rather than to measure sensitivity to DE gene availability
- When the dataset has no pre-identified DE genes or no ground-truth rare-cell labels,
  since F1 versus DE count cannot be computed
- When evaluating performance driven by non-DE signals (e.g. dropout-based Gini-type
  signals alone); this protocol explicitly removes and reintroduces DE genes only
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scCAD
  role: Rare-cell detector under test; run with rare_h = 0.01, median nearest-neighbor
    center distance merge threshold, and independence score threshold I = 0.7 on each
    DE-gene-substituted matrix
  repo: https://github.com/xuyp-csu/scCAD
- name: FiRE
  role: Comparison rare-cell detector run with default parameters on every substituted
    matrix
  repo: https://github.com/princethewinner/FiRE
- name: GapClust
  role: Comparison rare-cell detector run with default parameters on every substituted
    matrix
  repo: https://github.com/fabotao/GapClust
- name: GiniClust3
  role: Comparison rare-cell detector run with default parameters on every substituted
    matrix; stabilizes near F1 ~ 0.6 on the Jurkat data
  repo: https://github.com/rdong08/GiniClust3
- name: splatter
  role: R package used to generate the artificial scRNA-seq benchmark dataset with
    a 1% rare group and controlled DE parameters
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51891-9
    title: 'scCAD: Cluster decomposition-based anomaly detection for rare cell identification
      in single-cell expression data'
  source_content_hash: sha256:a9b4f7f6a32e8d6c762cf5669f061951ac95bbcad2f07fd7ccfa86660742ef8a
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
  - splatter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# de-gene-substitution-sensitivity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A robustness/sensitivity protocol that removes pre-identified differentially expressed (DE) genes from an scRNA-seq matrix and substitutes them back in controlled counts, then re-runs rare-cell detectors (scCAD vs FiRE, GapClust, GiniClust3) and plots average F1 score versus DE gene count.

## When to use

Use when quantifying how the number of differentially expressed genes available to a rare-cell identification method affects its F1 score, e.g. testing scCAD's robustness in weak differential expression scenarios against FiRE, GapClust, and GiniClust3.

## When NOT to use

- When the goal is simply to call rare cells on a real dataset with its full complement of DE genes rather than to measure sensitivity to DE gene availability
- When the dataset has no pre-identified DE genes or no ground-truth rare-cell labels, since F1 versus DE count cannot be computed
- When evaluating performance driven by non-DE signals (e.g. dropout-based Gini-type signals alone); this protocol explicitly removes and reintroduces DE genes only

## Inputs

- Artificial scRNA-seq dataset generated with splatter (group.prob = c(0.99, 0.01), batchCells = 2500, nGenes = 5000, de.prob = c(0.4, 0.4), de.facLoc = 0.4, de.facScale = 0.8, seed = 2023)
- Packaged 1%Jurkat.h5 subsampled dataset (1540 293T cells + 16 Jurkat cells, 32,738 genes) loaded from the scCAD repository
- Pre-identified DE gene lists per dataset (Wilcoxon rank sum, FDR 0.05)
- True rare-cell labels for F1 computation

## Outputs

- Table of average F1 scores per method per DE gene count (averaged over 10 random substitution runs)
- Plot of average F1 score versus DE gene count for scCAD, FiRE, GapClust, and GiniClust3 (qualitatively reproducing Supplementary Fig. 11)

## How to apply

First identify DE genes with a two-sided Wilcoxon rank sum test (FDR < 0.05; |fold-change| cutoff 1.5 for the artificial splatter data, 1 for the subsampled Jurkat data), yielding 220 reserved DE genes and 3226 non-DE genes for the artificial dataset and 108 DE genes and 30,479 non-DE genes for the 1%Jurkat dataset (1540 293T + 16 Jurkat cells), then remove all DE genes from the expression matrices. For each tested DE gene count, substitute an equal number of non-DE genes with randomly chosen pre-identified DE genes, repeating the random substitution 10 times per count. Run scCAD (rare_h = 0.01, merge threshold via median nearest-neighbor center distance, independence score threshold I = 0.7) plus FiRE, GapClust, and GiniClust3 with default parameters on every substituted matrix. Compute the F1 score of predicted versus true rare cells for each run, average across the 10 iterations per count, and plot average F1 versus DE gene count. The rationale is that removing and titrating DE genes isolates the contribution of the differential signal to detection performance, revealing that all methods fail with few DE genes but scCAD improves significantly and outperforms competitors as more DE genes are added.

## Related tools

- **scCAD** (Rare-cell detector under test; run with rare_h = 0.01, median nearest-neighbor center distance merge threshold, and independence score threshold I = 0.7 on each DE-gene-substituted matrix) — https://github.com/xuyp-csu/scCAD
- **FiRE** (Comparison rare-cell detector run with default parameters on every substituted matrix) — https://github.com/princethewinner/FiRE
- **GapClust** (Comparison rare-cell detector run with default parameters on every substituted matrix) — https://github.com/fabotao/GapClust
- **GiniClust3** (Comparison rare-cell detector run with default parameters on every substituted matrix; stabilizes near F1 ~ 0.6 on the Jurkat data) — https://github.com/rdong08/GiniClust3
- **splatter** (R package used to generate the artificial scRNA-seq benchmark dataset with a 1% rare group and controlled DE parameters)

## Examples

```
result, score, sub_clusters, degs_list = scCAD.scCAD(data=data, dataName='Jurkat', cellNames=cellNames, geneNames=geneNames, save_path='./')
```

## Evaluation signals

- Average-F1 curves qualitatively reproduce Supplementary Fig. 11: all methods fail with few DE genes and improve as more are added
- scCAD reaches higher F1 than FiRE, GapClust, and GiniClust3 in weak differential expression scenarios
- GiniClust3 stabilizes near F1 ~ 0.6 on the Jurkat substitution series
- Wilcoxon DE identification recovers the expected counts (220 DE / 3226 non-DE for artificial; 108 DE / 30,479 non-DE for Jurkat)
- Each F1 average spans exactly 10 random substitution repetitions per DE gene count

## Limitations

- All methods fail when few DE genes are available, so conclusions at low DE gene counts separate methods poorly
- Random substitution introduces sampling variability, mitigated only by 10 repetitions per count rather than exhaustive substitution
- The DE gene pool depends on the Wilcoxon fold-change cutoff, which must be adjusted per dataset (1.5 artificial vs 1 Jurkat); other cutoffs change the reserved DE gene set

## Evidence

- [discussion] independence score threshold I = 0.7: [evidence span withheld — anchor status: not_found]
- [discussion] iterative cluster decomposition threshold: [evidence span withheld — anchor status: not_found]
