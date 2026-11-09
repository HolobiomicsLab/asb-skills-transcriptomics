---
name: multi-order-neighbor-affinity-optimization
description: Use when adapting a pre-trained single-cell classifier to a target scRNA-seq/CITE-seq/scATAC-seq
  dataset with no access to source data, and batch effect must be mitigated self-supervised
  from target neighborhoods.
when_to_use_negative:
- Raw source data are available and explicit feature alignment (e.g. scArches-style
  surgery) is possible — scEMAIL's adaptation is designed for the source-free setting
  where privacy or storage prevents sharing raw source data.
- Target data have no novel cell types and no source-target batch effect — the local
  affinity regularization and the D/Dt machinery add unnecessary complexity.
- Labels are unavailable even in a source dataset (this is a transfer/annotation method
  that requires a trained source classifier, not a de novo clustering method).
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scEMAIL
  role: Provides the source-free adaptation framework in which the multi-order local
    affinity loss over representation and soft-label memory banks is imposed on the
    ZINB autoencoder encoder h.
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:d93c581f0ac34d2f051a95ea74cc42fa6a91a1fcb8a9b5bb4ae95de833dca73b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# multi-order-neighbor-affinity-optimization

## Summary

A source-free model-adaptation step from scEMAIL that fine-tunes only the ZINB autoencoder encoder h (classifier weights fixed) by minimizing target ZINB denoising NLL plus a local affinity loss built from bidirectional, unidirectional, and extensional nearest neighbors, to implicitly mitigate source-target batch effect without raw source data.

## When to use

Use when adapting a pre-trained single-cell classifier to a target scRNA-seq/CITE-seq/scATAC-seq dataset with no access to source data, and batch effect must be mitigated self-supervised from target neighborhoods.

## When NOT to use

- Raw source data are available and explicit feature alignment (e.g. scArches-style surgery) is possible — scEMAIL's adaptation is designed for the source-free setting where privacy or storage prevents sharing raw source data.
- Target data have no novel cell types and no source-target batch effect — the local affinity regularization and the D/Dt machinery add unnecessary complexity.
- Labels are unavailable even in a source dataset (this is a transfer/annotation method that requires a trained source classifier, not a de novo clustering method).

## Inputs

- Pre-trained scEMAIL source model weights (encoder h, classifier c, auxiliary classifiers c1/c2)
- Target single-cell count matrix (scRNA-seq, 10x Genomics 10k PBMC CITE-seq, or mouse bone marrow scATAC-seq)
- Known/unknown cell partition D/Dt from the E-score threshold δ (manifold mixup), when novel cell types are detected

## Outputs

- Fine-tuned encoder h adapted to the target distribution
- Per-cell annotation probabilities from classifier c (argmax labels for cells in D; 'unknown' for Dt)

## How to apply

Load the pre-trained source model (ZINB autoencoder encoder h, main classifier c, auxiliary classifiers c1/c2) and freeze the classifier weights; fine-tune only h on the target count matrices. Each iteration, retrieve M-nearest neighbors per cell by cosine similarity from two memory banks — one over cell representations and one over soft labels — including the cell itself. Weight neighbors bidirectionally at full weight, unidirectionally by α = 0.1, and add extensional neighbors; restrict the neighbor search to cells in the 'known' set D (E-score > δ from manifold mixup) when novel cell types are present, and refresh the D/Dt partition every iteration. Minimize the ZINB denoising negative log-likelihood plus the multi-order local affinity regularization so reliable neighbor soft labels provide self-supervised signal that corrects wrong source-model classifications without feature alignment to source data.

## Related tools

- **scEMAIL** (Provides the source-free adaptation framework in which the multi-order local affinity loss over representation and soft-label memory banks is imposed on the ZINB autoencoder encoder h.) — https://github.com/aster-ww/scEMAIL

## Examples

```
python scemail_adapt.py --source_model pretrained_scemail.pt --target pbmc_cite_seq_counts.h5 --neighbors M --unidirectional_alpha 0.1 --freeze_classifier --output scemail_crossmodality_annotations.csv
```

## Evaluation signals

- Annotation accuracy/H-score on the adapted target (e.g. 10x 10k PBMC CITE-seq, mouse bone marrow scATAC-seq) matches the paper's cross-modality results shown in Figure 10.
- Cells in the 'known' set D receive argmax labels from classifier c while cells in Dt are all labeled 'unknown' — check the output CSV has one row per target cell and an 'unknown' category where applicable.
- Classifier weights (c, c1, c2) remain unchanged after adaptation; only encoder h parameters update.
- Neighbor lists contain exactly M entries per cell, retrieved by cosine similarity, include the cell itself, and never draw from Dt when novel cell types are present.
- H-scores in CITE-seq leave-one-out experiments improve by 2–6 percentage points when surface protein measurements are added to the network input.

## Limitations

- The novel-type split D/Dt and threshold δ depend on the E-score distribution being bimodal (bimodality coefficient / Hartigan's dip test); unimodal contamination may be missed.
- Unidirectional neighbors are down-weighted to α = 0.1, so cells with few reliable bidirectional neighbors receive weaker self-supervised correction.
- Annotation accuracy values for the scATAC-seq 'mouse bone marrow' experiment are only shown in Figure 10, not reported numerically in the text, complicating direct numeric validation.
- Detailed implementation parameters are deferred to supplementary File S1 rather than the main text.

## Evidence

- [intro] we search for bidirectional, unidirectional, and extensional neighbors of each target cell in global scope. Based on the gathered multi-order neighborhood message, we impose various local affinity: "we search for bidirectional, unidirectional, and extensional neighbors of each target cell in global scope. Based on the gathered multi-order neighborhood message, we impose various local affinity"
- [intro] the denoising ZINB loss and multi-order local affinity constraints are imposed to implicitly mitigate batch effect between source and target data.: "the denoising ZINB loss and multi-order local affinity constraints are imposed to implicitly mitigate batch effect between source and target data."
- [methods] we further introduce two auxiliary classifiers cj, j = 1, 2, which also take features from h for further uncertainty measurement: "we further introduce two auxiliary classifiers cj, j = 1, 2, which also take features from h for further uncertainty measurement"
- [readme] These constraints mitigate wrong classifications of the source model via reliable self-supervised information of neighbors.: "These constraints mitigate wrong classifications of the source model via reliable self-supervised information of neighbors."
