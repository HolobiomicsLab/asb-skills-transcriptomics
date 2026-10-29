---
name: single-cell-cell-type-annotation-transfer
description: Use when raw source scRNA-seq data are unavailable (privacy/storage)
  but pre-trained source model weights exist, and target data may contain novel cell
  types requiring source-free annotation.
when_to_use_negative:
- Raw source data are available and explicit source-target feature alignment is possible
  (other transfer methods may suffice without scEMAIL's source-free constraint)
- A subjective/user-defined threshold for novel cell detection is acceptable and the
  task is simple clustering rather than label transfer from a trained source model
- The target data are not single-cell count data (e.g. bulk RNA-seq or bulk ATAC-seq),
  where the ZINB count model and per-cell E-score machinery do not apply
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scEMAIL
  role: 'Core framework: ZINB autoencoder-based source-free annotation with expert
    ensemble novel cell-type perception and multi-order local affinity constraints'
  repo: https://github.com/aster-ww/scEMAIL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2022.12.008
    title: 'scEMAIL: Universal and Source-Free Annotation Method for scRNA-Seq Data
      with Novel Cell-Type Perception'
  source_content_hash: sha256:0d18aaaabbfa9c19bb78a9239b75b72a84dc519e7719b9ca6d40bee85191923e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aster-ww/scEMAIL
  tools:
  - scEMAIL
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# single-cell-cell-type-annotation-transfer

## Summary

Source-free transfer learning annotation of single-cell target data using scEMAIL: only pre-trained source model weights (ZINB autoencoder encoder h, main classifier c, two auxiliary classifiers c1/c2) are shared, and target cells are annotated with automatic detection of novel cell types via an expert ensemble E-score, bimodality tests, manifold-mixup thresholding, and multi-order local affinity constraints.

## When to use

Use when raw source scRNA-seq data are unavailable (privacy/storage) but pre-trained source model weights exist, and target data may contain novel cell types requiring source-free annotation.

## When NOT to use

- Raw source data are available and explicit source-target feature alignment is possible (other transfer methods may suffice without scEMAIL's source-free constraint)
- A subjective/user-defined threshold for novel cell detection is acceptable and the task is simple clustering rather than label transfer from a trained source model
- The target data are not single-cell count data (e.g. bulk RNA-seq or bulk ATAC-seq), where the ZINB count model and per-cell E-score machinery do not apply

## Inputs

- Pre-trained scEMAIL source model weights (encoder h of ZINB autoencoder, main classifier c, auxiliary classifiers c1 and c2)
- Target scRNA-seq count matrix (e.g. 10x Genomics 10k PBMCs CITE-seq gene expression counts)
- Optional paired cell surface protein measurements (CITE-seq ADT counts) to include in the network input
- Optional scATAC-seq count matrix (e.g. mouse bone marrow scATAC-seq)

## Outputs

- Per-cell annotation labels CSV (e.g. scemail_crossmodality_annotations.csv) with one row per target cell
- 'unknown' category assignments for cells in the unknown set Dt
- Known/unknown partition (sets D and Dt) refreshed each adaptation iteration

## How to apply

Load the pre-trained scEMAIL source model (ZINB autoencoder encoder h, main classifier c, and two randomly initialized auxiliary classifiers c1/c2) and apply it to the target count matrix. Compute a per-cell expert ensemble E-score as the average of (1 − EN/log K), maximum softmax confidence CF, and consistency CS (dot product of the two auxiliary probability vectors); test the E-score distribution for bimodality using the bimodality coefficient and Hartigan's dip test. If bimodal, construct synthetic representations by manifold mixup (λ ~ Uniform(0,1) over all pairs of target latent vectors), set threshold δ to the average E-score over all pairs, and partition cells into known set D (E-score > δ) and unknown set Dt. Adapt source-free by fixing classifier weights and fine-tuning only encoder h with the target ZINB denoising negative log-likelihood plus a local affinity loss over M nearest neighbors (cosine similarity, soft-label memory banks including the cell itself), weighting bidirectional neighbors fully, unidirectional neighbors by α = 0.1, adding extensional neighbors, restricting neighbors to cells in D, and refreshing D/Dt each iteration. Output argmax labels from classifier c for cells in D and 'unknown' for Dt. The framework is also flexibly applicable to CITE-seq (adding cell surface protein measurements to the network input) and scATAC-seq data.

## Related tools

- **scEMAIL** (Core framework: ZINB autoencoder-based source-free annotation with expert ensemble novel cell-type perception and multi-order local affinity constraints) — https://github.com/aster-ww/scEMAIL

## Examples

```
labels = scemail.annotate(source_model='pretrained_scemail.pt', target='10k_PBMC_CITEseq.h5', use_protein=True, alpha=0.1, output='scemail_crossmodality_annotations.csv')
```

## Evaluation signals

- Output CSV exists with exactly one row per target cell and contains an 'unknown' category where bimodality tests indicated novel cell types
- Bimodality coefficient or Hartigan's dip test flags bimodal structure in the empirical E-score distribution when target data contain novel cell types
- Reproduced annotation accuracy/H-score matches values reported for the CITE-seq and scATAC-seq experiments (e.g. Figure 10 of the paper)
- In CITE-seq leave-one-out experiments on 10x Genomics 10k PBMCs, H-scores increase by two to six percentage points when protein measurements are added to the network input
- Unidirectional neighbors are weighted by α = 0.1 and only cells in the known set D are used as neighbors during fine-tuning

## Limitations

- Annotation accuracy for the scATAC-seq 'mouse bone marrow' experiment is only shown graphically in Figure 10, not reported numerically in the text
- Implementation details for competing methods (Table S1), simulated experiments (File S1), and scalability/robustness results (File S1) are deferred to supplementary materials
- Extension to paired CITE-seq and scATAC-seq data is demonstrated only preliminarily ('Some attempts on the paired CITE-seq data and scATAC-seq data in the experiment section can preliminarily illustrate this statement.')

## Evidence

- [abstract] We propose a universal annotation framework for scRNA-seq data called scEMAIL, which automatically detects novel cell types without accessing source data during adaptation.: "We propose a universal annotation framework for scRNA-seq data called scEMAIL, which automatically detects novel cell types without accessing source data during adaptation."
- [methods] we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS).: "we adopt an expert ensemble system including three components: entropy (EN), confidence (CF), and consistency (CS)."
- [readme] It is also flexible to be applied to challenging single-cell ATAC-seq data without loss of superiority.: "It is also flexible to be applied to challenging single-cell ATAC-seq data without loss of superiority."
- [abstract] an adaptive threshold via manifold mixup partitions target cells into "known" and "unknown" groups.: "an adaptive threshold via manifold mixup partitions target cells into "known" and "unknown" groups."
- [discussion] Some attempts on the paired CITE-seq data and scATAC-seq data in the experiment section can preliminarily illustrate this statement.: "Some attempts on the paired CITE-seq data and scATAC-seq data in the experiment section can preliminarily illustrate this statement."
