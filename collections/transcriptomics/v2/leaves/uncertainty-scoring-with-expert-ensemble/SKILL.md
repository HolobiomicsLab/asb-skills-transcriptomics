---
name: uncertainty-scoring-with-expert-ensemble
description: Use when annotating target scRNA-seq cells with a pre-trained source-free
  classifier and you need per-cell uncertainty scores to detect whether novel cell
  types are present without manually defined thresholds.
when_to_use_negative:
- Raw source data are available and explicit source-target feature alignment is possible
  (standard transfer learning methods apply)
- Only a single classifier is available — the consistency component requires two auxiliary
  classifiers with different random initializations
- Labels for all target cells are already known and no novel-type screening is needed
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEMAIL
  role: Implements the expert ensemble system (entropy, confidence, consistency) and
    downstream bimodality tests within the source-free annotation framework
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:f49e195a6858e96cc954a1e79f2ca8510cd66c32b5b3fdd477252f50c89fa673
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
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# uncertainty-scoring-with-expert-ensemble

## Summary

Expert ensemble uncertainty scoring from scEMAIL: each target scRNA-seq cell receives an E-score combining normalized entropy, maximum confidence, and consistency between two auxiliary co-training classifiers, enabling source-free novel cell-type detection.

## When to use

Use when annotating target scRNA-seq cells with a pre-trained source-free classifier and you need per-cell uncertainty scores to detect whether novel cell types are present without manually defined thresholds.

## When NOT to use

- Raw source data are available and explicit source-target feature alignment is possible (standard transfer learning methods apply)
- Only a single classifier is available — the consistency component requires two auxiliary classifiers with different random initializations
- Labels for all target cells are already known and no novel-type screening is needed

## Inputs

- Pre-trained source classification model weights (ZINB autoencoder encoder + main classifier + two auxiliary classifiers)
- Target scRNA-seq expression matrix (cells x genes) without raw source data
- Number of known source cell types K

## Outputs

- Per-cell expert ensemble E-scores for all N target cells
- Bimodality test verdicts (bimodality coefficient and Hartigan's dip test) on the E-score distribution
- Binary decision on the presence of novel cell types in the target data

## How to apply

Train the source model with a ZINB autoencoder plus a supervised cross-entropy classifier and two auxiliary classifiers with different random initializations. For each target cell, compute the E-score as the average of three components: normalized entropy (EN/logK), maximum confidence (CF), and consistency (CS), defined as the dot product of the two auxiliary classifiers' soft probability vectors. Consistency is robust to batch-effect-induced prediction errors because both classifiers are unlikely to misclassify a cell into the same type. Test the empirical E-score distribution for bimodality using the bimodality coefficient and Hartigan's dip test; a bimodal structure indicates novel cell types. The E-score then feeds the adaptive manifold-mixup threshold that partitions target cells into 'known' and 'unknown' groups.

## Related tools

- **scEMAIL** (Implements the expert ensemble system (entropy, confidence, consistency) and downstream bimodality tests within the source-free annotation framework) — https://github.com/aster-ww/scEMAIL

## Examples

```
git clone https://github.com/aster-ww/scEMAIL && python run_scEMAIL.py --task bone_marrow_closed --source_model pretrained_source.pth --target target.h5ad
```

## Evaluation signals

- E-score distributions of target data containing novel cell types should exhibit an apparent bimodal structure
- At least one of the bimodality coefficient or Hartigan's dip test must indicate bimodality to declare novel cell types
- Consistency scores should remain stable under batch effect, since both auxiliary classifiers rarely mispredict the same cell type
- Reproduced benchmark accuracies after threshold-based partitioning should match reported values (e.g., bone marrow 0.8873, trachea 0.9415) with scEMAIL ranking highest

## Limitations

- Requires two auxiliary classifiers trained from the source model; not applicable with single-classifier pipelines
- Novel-type detection relies on bimodality of the E-score distribution and may fail if novel and known cells are not separable in uncertainty
- The adaptive threshold depends on manifold-mixup pairs of target cells; very small target datasets may give unstable thresholds

## Evidence

- [methods] expert ensemble components: "we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS)."
- [methods] consistency robustness rationale: "Consistency is robust to prediction errors caused by the batch effect, as it is less likely that both classifiers will wrongly predict a cell into exactly the same cell type."
- [methods] bimodality decision rule: "As long as one of these measures indicates the bimodal structure of data distribution, we will suppose the existence of novel cell types in the following training process."
- [methods] bimodal E-score structure finding: "When target data have novel cell types, it is interesting that their empirical distribution of E-score(zit)i=1Nt exhibits apparent bimodal structure"
- [readme] skill motivation from README: "an expert ensemble system measures uncertainty of each cell from three complementary aspects. Second, based on this measurement, bimodality tests are applied to detect the presence of new cell types."
