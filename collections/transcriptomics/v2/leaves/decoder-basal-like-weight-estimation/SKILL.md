---
name: decoder-basal-like-weight-estimation
description: Use when a malignant epithelial subcluster (e.g. from PDAC scRNA-seq)
  needs molecular subtype assignment via pathway deconvolution, specifically estimating
  basal-like vs classical pathway weights, bcRatio, and GATA6 status.
when_to_use_negative:
- The cell population is already unambiguously classified as classical or basal-like
  and only marker plots are needed — DECODER deconvolution adds nothing over direct
  marker scoring.
- The input contains no malignant epithelial cells (e.g. stromal/immune-only clusters);
  DECODER subtype weights are designed for tumor cell populations, as in the PDAC
  ductal/PDAC subsetting use case.
- No subtype gene signatures are available or the dataset is not human PDAC-compatible
  with the Moffitt/DECODER gene sets.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: decoderr
  role: R implementation of DECODER used to estimate basal-like pathway weights and
    compute the bcRatio per tumor subcluster
  repo: https://github.com/laurapeng/decoderr
- name: SCISSORS
  role: Produces the rare basal-like tumor subcluster (via silhouette scoring and
    multi-step semi-supervised reclustering) whose weights DECODER then estimates
  repo: https://github.com/jr-leary7/SCISSORS
- name: CONICSmat
  role: Detects aberrant copy-number variations in the reclustered cells to confirm
    malignant identity of the subcluster being scored
  repo: https://github.com/Neurosurgery-Brain-Tumor-Center-DiazLab/CONICS
- name: Seurat
  role: Hosts the scRNA-seq object and provides the expression data and differential
    marker expression used for the GATA6 depletion check
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:540f009f8b8931a6f0e3a74e8b931a2b77fae80145b11507582acfe795e0d7af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/laurapeng/decoderr
  tools:
  - CONICSmat
  - DECODER
  - R
  - SCISSORS
  - Seurat
  - decoderr
  - dplyr
  - ggplot2
  - magrittr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# decoder-basal-like-weight-estimation

## Summary

Applies the DECODER deconvolution method (R package 'decoderr') to scRNA-seq tumor subclusters — e.g. PDAC epithelial cells recovered by SCISSORS reclustering — to estimate per-subcluster basal-like and classical pathway weights, compute the bcRatio, and verify GATA6 depletion, thereby assigning molecular subtype identity to rare malignant subpopulations.

## When to use

Use when a malignant epithelial subcluster (e.g. from PDAC scRNA-seq) needs molecular subtype assignment via pathway deconvolution, specifically estimating basal-like vs classical pathway weights, bcRatio, and GATA6 status.

## When NOT to use

- The cell population is already unambiguously classified as classical or basal-like and only marker plots are needed — DECODER deconvolution adds nothing over direct marker scoring.
- The input contains no malignant epithelial cells (e.g. stromal/immune-only clusters); DECODER subtype weights are designed for tumor cell populations, as in the PDAC ductal/PDAC subsetting use case.
- No subtype gene signatures are available or the dataset is not human PDAC-compatible with the Moffitt/DECODER gene sets.

## Inputs

- Normalized scRNA-seq count/expression matrix (e.g. Seurat object assay data) for tumor/epithelial cells
- Subcluster labels for the cells of interest (e.g. from SCISSORS semi-supervised reclustering of combined epithelial Clusters 5 and 9)
- Basal-like and classical gene signatures (e.g. Moffitt basal-like and classical gene sets)

## Outputs

- Per-subcluster DECODER basal-like and classical pathway weights
- bcRatio per subcluster (basal-like vs classical weight ratio)
- Differential expression result confirming GATA6 depletion in the basal-like subcluster

## How to apply

Run DECODER on the expression matrix of the reclustered tumor cells (e.g. the combined Elyada PDAC epithelial Clusters 5 and 9 after SCISSORS subclustering with PrepareData n.HVG=4000, n.PC=15, initial.resolution=0.4, random.seed=629) to estimate pathway weights for the basal-like and classical signatures. Compute the bcRatio (basal-like weight divided by classical weight) per subcluster to quantify subtype skew. Cross-check the weights against an independent signature-scoring method such as VAM with the Moffitt basal-like and classical gene sets — the correct outcome is high basal-like weights with absence of classical signal. Finally, verify GATA6 depletion in the basal-like subcluster through differential marker expression, since GATA6 loss is the expected epithelial plasticity marker accompanying basal-like identity. In the reference application, the recovered basal-like subcluster comprised 148 cells (0.59% of the dataset), showed strong CONICSmat CNV aberrations confirming malignancy, and exhibited high basal-like DECODER weights and bcRatio with GATA6 depletion.

## Related tools

- **decoderr** (R implementation of DECODER used to estimate basal-like pathway weights and compute the bcRatio per tumor subcluster) — https://github.com/laurapeng/decoderr
- **SCISSORS** (Produces the rare basal-like tumor subcluster (via silhouette scoring and multi-step semi-supervised reclustering) whose weights DECODER then estimates) — https://github.com/jr-leary7/SCISSORS
- **CONICSmat** (Detects aberrant copy-number variations in the reclustered cells to confirm malignant identity of the subcluster being scored) — https://github.com/Neurosurgery-Brain-Tumor-Center-DiazLab/CONICS
- **Seurat** (Hosts the scRNA-seq object and provides the expression data and differential marker expression used for the GATA6 depletion check)

## Evaluation signals

- The target subcluster shows high basal-like DECODER pathway weights and a high bcRatio, with absence of classical weights.
- VAM scoring of Moffitt basal-like and classical signatures independently confirms basal-like enrichment and classical-signature absence in the same subcluster.
- GATA6 expression is depleted in the basal-like subcluster relative to classical tumor cells via differential marker expression.
- CONICSmat reports strong CNV aberrations in the subcluster, supporting its malignant (not normal ductal) identity.
- In the reference reproduction, the basal-like subcluster contains 148 cells, i.e. 0.59% of the Elyada PDAC dataset — a rare subpopulation consistent with SCISSORS' rare-cell optimization.

## Limitations

- DECODER weights depend on the quality of upstream subclustering: even small differences in analysis and parameter choice can greatly alter clustering results, which can change which cells enter the subcluster being scored.
- Subtype weight interpretation is anchored to the provided basal-like/classical gene signatures (e.g. Moffitt); datasets or species outside the signature's scope may not be interpretable.
- Basal-like calls should be triangulated with orthogonal evidence (CNVs via CONICSmat, GATA6 depletion, signature scoring) — DECODER weights alone are insufficient.

## Evidence

- [other] Run DECODER (decoderr) to compute basal-like pathway weights and the bcRatio, and verify GATA6 depletion through differential marker expression.: [evidence span withheld — anchor status: not_found]
- [other] high basal-like DECODER weights and bcRatio, and GATA6 depletion: [evidence span withheld — anchor status: not_found]
- [abstract] SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [readme] Tools are included to preprocess data, identify potential targets for subgroup analysis, optimize reclustering, and annotate results.: "Tools are included to preprocess data,
identify potential targets for subgroup analysis, optimize reclustering,
and annotate results."
