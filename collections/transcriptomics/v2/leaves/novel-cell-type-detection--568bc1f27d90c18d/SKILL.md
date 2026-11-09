---
name: novel-cell-type-detection
description: Use when a pre-trained source model must annotate target scRNA-seq cells
  without access to source data, and the target may contain cell types absent from
  the source label set, with no user-specified threshold allowed.
when_to_use_negative:
- The target data are known to contain only cell types present in the source label
  set and a bimodality test of the E-score distribution is not triggered — then all
  cells belong to the known group and no thresholding is needed
- Raw source data are available and you want explicit source-target feature alignment
  (e.g. ItClust-style transfer) instead of source-free adaptation
- Input is not single-cell omics data with a pre-trained source classifier providing
  soft probability vectors (e.g. bulk RNA-seq or feature tables without per-cell predictions)
tools:
- name: scEMAIL
  role: Implements the expert ensemble E-score, bimodality tests (bimodality coefficient
    and Hartigan's dip test), and manifold-mixup adaptive threshold for novel cell-type
    perception
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:01699c345712f1f92e2afbac9f796ff08ad16104a2bb1ed18faa22b954c8bc5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - ItClust
  - Seurat v3
  - scArches
  - scEMAIL
  - scSemiCluster
  - scmap
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# novell-cell-type-detection

## Summary

Detect novel cell types in target scRNA-seq data source-free by scoring each cell with an expert-ensemble E-score (entropy, confidence, consistency), testing the E-score distribution for bimodality, and partitioning cells into 'known' and 'unknown' groups via an adaptive manifold-mixup threshold. This is scEMAIL's novel cell-type perception module (Wan et al., GPB 2022).

## When to use

Use when a pre-trained source model must annotate target scRNA-seq cells without access to source data, and the target may contain cell types absent from the source label set, with no user-specified threshold allowed.

## When NOT to use

- The target data are known to contain only cell types present in the source label set and a bimodality test of the E-score distribution is not triggered — then all cells belong to the known group and no thresholding is needed
- Raw source data are available and you want explicit source-target feature alignment (e.g. ItClust-style transfer) instead of source-free adaptation
- Input is not single-cell omics data with a pre-trained source classifier providing soft probability vectors (e.g. bulk RNA-seq or feature tables without per-cell predictions)

## Inputs

- Latent representations z_it of target cells from the pre-trained source model's ZINB autoencoder encoder h
- Soft probability vectors from the main classifier c (K-dimensional) for every target cell
- Soft probability vectors from the two auxiliary classifiers c1 and c2 for every target cell

## Outputs

- Per-cell E-scores (e_scores.csv), each within [0,1]
- Per-cell 'known'/'unknown' group assignments together with the threshold value (threshold_partition.csv)
- Final labels: classifier argmax for D cells, 'unknown' for D_t cells

## How to apply

Load the latent representations z_it and soft probability vectors from the source model's main classifier c and two randomly initialized auxiliary classifiers c1/c2. Compute the expert ensemble E-score E(z_it) = (1 - EN(z_it)/log K + CF(z_it) + CS(z_it))/3, where EN is normalized entropy of the main classifier's K-dimensional prediction, CF = max_k p_ik, and CS is the element-wise product of the two auxiliary classifiers' probability vectors; consistency is robust to batch-effect-induced errors since both classifiers are unlikely to make the same wrong prediction. Test the empirical E-score distribution for bimodality using the bimodality coefficient and Hartigan's dip test; if neither indicates bimodality, assign all target cells to the known group and skip thresholding. If bimodality is confirmed, construct manifold-mixup synthetic representations z~ = lambda*z_i + (1-lambda)*z_j with lambda ~ Uniform(0,1) for all target-cell pairs, set the adaptive threshold delta to the mean E-score over all pairs, and label cells with E-score > delta as 'known' (D) and the rest as 'unknown' (D_t). Write per-cell E-scores and group assignments so downstream adaptation (frozen classifiers, fine-tuned encoder with local affinity losses) can use D as reliable self-supervision and D_t as the 'unknown' label.

## Related tools

- **scEMAIL** (Implements the expert ensemble E-score, bimodality tests (bimodality coefficient and Hartigan's dip test), and manifold-mixup adaptive threshold for novel cell-type perception) — https://github.com/aster-ww/scEMAIL

## Examples

```
git clone https://github.com/aster-ww/scEMAIL && python scemail.py --source_model pretrained_source.pt --target target.h5ad --e_scores e_scores.csv --partition threshold_partition.csv
```

## Evaluation signals

- e_scores.csv exists and every E-score value falls within [0,1], consistent with the normalized entropy, confidence, and consistency components
- Number of known + unknown cells equals Nt, and every cell with E-score > delta is labeled 'known', exactly matching the partition rule
- When target data contain novel cell types, the empirical E-score distribution exhibits apparent bimodal structure per the bimodality coefficient or Hartigan's dip test
- If neither bimodality test is positive, the known group equals all Nt target cells and D_t is empty
- Downstream: cells in D receive the argmax label of classifier c and cells in D_t are labeled 'unknown'

## Limitations

- If the bimodality coefficient and Hartigan's dip test both fail to detect bimodality, novel cell types will not be perceived and all target cells will be treated as known
- Details of implementation (e.g. simulation setup, scalability experiments) are deferred to supplementary File S1 and are not in the article text
- The method is demonstrated on scRNA-seq and extended to CITE-seq and scATAC-seq; scATAC-seq accuracy is only shown in Figure 10, not reported numerically in the text

## Evidence

- [methods] Expert ensemble system for uncertainty measurement: "we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS)."
- [methods] Adaptive threshold via manifold mixup: "The final threshold can be adaptively and automatically decided by taking the average E-score of all possible pairs of the target cells"
- [methods] Bimodality test decision rule: "As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process."
- [methods] E-score bimodality when novel types present: "When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
- [readme] Module overview from README: "once assured of their presence, an adaptive threshold via manifold mixup partitions target cells into “known” and “unknown” groups."
