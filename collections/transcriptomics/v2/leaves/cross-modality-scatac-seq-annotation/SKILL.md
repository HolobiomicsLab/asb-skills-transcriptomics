---
name: cross-modality-scatac-seq-annotation
description: Use when the target data are single-cell multi-omic counts (10x Genomics
  PBMC CITE-seq with antibody-derived tags, or scATAC-seq such as mouse bone marrow)
  and only pre-trained scEMAIL source weights (encoder h, classifier c, auxiliary
  c1/c2) are available.
when_to_use_negative:
- The target data are standard scRNA-seq — use the base scEMAIL annotation skill rather
  than the cross-modality extension
- Raw source data are available and explicit feature alignment between source and
  target is desired (e.g., scArches-style mapping), since scEMAIL is designed for
  the source-free setting
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scEMAIL
  role: Source-free annotation framework extended to CITE-seq and scATAC-seq targets
    via expert ensemble E-score, bimodality tests, manifold mixup thresholding, and
    ZINB + local affinity adaptation
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:b9da6c2e64cce52cc1a0367c3813833974094a9252a54864387fbb9549832e1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cross-modality-scatac-seq-annotation

## Summary

Extension of the scEMAIL source-free annotation framework beyond scRNA-seq to paired CITE-seq (gene expression + cell surface proteins) and scATAC-seq count matrices, by feeding the multi-modal network input into the pre-trained ZINB autoencoder and re-running expert ensemble uncertainty, bimodality testing, manifold-mixup thresholding, and local-affinity model adaptation.

## When to use

Use when the target data are single-cell multi-omic counts (10x Genomics PBMC CITE-seq with antibody-derived tags, or scATAC-seq such as mouse bone marrow) and only pre-trained scEMAIL source weights (encoder h, classifier c, auxiliary c1/c2) are available.

## When NOT to use

- The target data are standard scRNA-seq — use the base scEMAIL annotation skill rather than the cross-modality extension
- Raw source data are available and explicit feature alignment between source and target is desired (e.g., scArches-style mapping), since scEMAIL is designed for the source-free setting

## Inputs

- Pre-trained scEMAIL source model weights (ZINB autoencoder encoder h, main classifier c, auxiliary classifiers c1 and c2)
- Target count matrices: 10x Genomics 10k PBMCs CITE-seq dataset (gene expression plus cell surface protein measurements)
- Target count matrices: mouse bone marrow scATAC-seq dataset

## Outputs

- Per-cell annotation CSV (scemail_crossmodality_annotations.csv) with one row per target cell, including an 'unknown' category for cells in Dt
- Reproduced cross-modality annotation accuracy / H-scores comparable to the paper's Figure 10 values

## How to apply

Load the pre-trained scEMAIL source model (ZINB autoencoder encoder h, main classifier c, two randomly initialized auxiliary classifiers c1/c2) and the target count matrices (e.g., 10x Genomics 10k PBMCs CITE-seq or mouse bone marrow scATAC-seq). For every target cell compute the E-score as the average of (1 − EN/log K), the maximum softmax confidence CF, and the consistency CS (dot product of the two auxiliary probability vectors); test its empirical distribution for bimodality with the bimodality coefficient and Hartigan's dip test. If bimodal, construct synthetic representations via manifold mixup (λ ~ Uniform(0,1) over all target latent pairs) and set threshold δ to the average E-score over all pairs, partitioning cells into known set D (E-score > δ) and unknown set Dt; otherwise D is all cells. Adapt source-free by fixing the classifier weights and fine-tuning only encoder h on the target ZINB denoising negative log-likelihood plus a local affinity loss over M-nearest neighbors (cosine similarity from representation and soft-label memory banks, including the cell itself), weighting bidirectional neighbors fully and unidirectional neighbors by α = 0.1, with extensional neighbors added and neighbors restricted to cells in D, refreshing D and Dt each iteration. Adding cell surface protein measurements to the network input improved H-scores by two to six percentage points in all five CITE-seq leave-one-out experiments, demonstrating that the modality extension does not sacrifice performance. Final labels come from the argmax of c for cells in D and 'unknown' for cells in Dt.

## Related tools

- **scEMAIL** (Source-free annotation framework extended to CITE-seq and scATAC-seq targets via expert ensemble E-score, bimodality tests, manifold mixup thresholding, and ZINB + local affinity adaptation) — https://github.com/aster-ww/scEMAIL

## Examples

```
python scemail_annotate.py --source_model pretrained_scemail.pt --target 10k_pbmc_cite_seq.h5 --modality cite_seq --output scemail_crossmodality_annotations.csv
```

## Evaluation signals

- Output CSV exists with exactly one row per target cell and an explicit 'unknown' label where bimodality tests indicated novel cell types
- In CITE-seq leave-one-out experiments, adding cell surface protein measurements to the network input should raise H-scores by two to six percentage points in all five experiments
- Reproduced annotation accuracies for the CITE-seq and scATAC-seq experiments match the values reported/displayed in Figure 10
- The bimodality coefficient and Hartigan's dip test agree on whether novel cell types are assumed present before thresholding
- Model adaptation fixes classifier weights and updates only encoder h, with neighbors restricted to cells in the known set D and refreshed every iteration

## Limitations

- Annotation accuracy values for the scATAC-seq 'mouse bone marrow' experiment are only shown in Figure 10 and are not reported numerically in the text
- Detailed biological and statistical information about the cross-tissue annotation tasks is deferred to supplementary Table S5; implementation details of competing methods are in Table S1
- Novel-type detection relies on indirect bimodality tests (bimodality coefficient, Hartigan's dip test) rather than prior information or user-defined thresholds

## Evidence

- [abstract] It is also flexible to be applied to challenging single-cell ATAC-seq data without loss of superiority.: "It is also flexible to be applied to challenging single-cell ATAC-seq data without loss of superiority."
- [results] H-scores increasing by two to six percentage points: [evidence span withheld — anchor status: not_found]
- [methods] we adopt an expert ensemble system including three components: "we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS)."
- [methods] The final threshold can be adaptively and automatically decided: "The final threshold can be adaptively and automatically decided by taking the average E-score of all possible pairs of the target cells"
- [readme] The source code of scEMAIL can be accessed at: "We propose a universal annotation framework for scRNA-seq data called scEMAIL, which automatically detects novel cell types without accessing source data during adaptation."
