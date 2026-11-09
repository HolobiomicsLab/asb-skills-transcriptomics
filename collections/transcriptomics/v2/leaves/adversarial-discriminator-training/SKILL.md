---
name: adversarial-discriminator-training
description: Use when training scButterfly-B (or -T/-C) on paired single-cell multi-omics
  count matrices to align modality-specific VAE latent spaces via translator–discriminator
  adversarial optimization before cross-modality prediction.
when_to_use_negative:
- When you only need per-modality reconstruction or embedding without cross-modality
  translation — plain VAE pretraining suffices and adversarial training adds cost
- When working with modalities not supported by scButterfly (e.g. spatial transcriptomics
  or single-cell Hi-C), which the paper lists as future directions only
- When no model has been constructed/preprocessed — train_model requires load_data,
  data_preprocessing, and construct_model to have run first
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scButterfly
  role: Provides Butterfly.train_model, construct_model, and test_model implementing
    the adversarial translator–discriminator training within the dual-aligned VAE
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment required for the scButterfly package (conda create scButterfly
    python==3.9)
- name: PyTorch
  role: Deep-learning backend for the encoders, translators, and sigmoid-output discriminators
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:d1136ec9c771ffd0c407abf4f7828f95647e8ba04f01eefba946ef4e0a5699a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - PyTorch
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# adversarial-discriminator-training

## Summary

Trains the scButterfly dual-aligned VAE by jointly optimizing translator and discriminator adversarial losses after per-modality autoencoder pretraining, so that cross-modality translated profiles align in latent space. This adversarial phase is what drives the dual-aligned latent representations used for scRNA-seq ↔ scATAC-seq (and scADT-seq) translation.

## When to use

Use when training scButterfly-B (or -T/-C) on paired single-cell multi-omics count matrices to align modality-specific VAE latent spaces via translator–discriminator adversarial optimization before cross-modality prediction.

## When NOT to use

- When you only need per-modality reconstruction or embedding without cross-modality translation — plain VAE pretraining suffices and adversarial training adds cost
- When working with modalities not supported by scButterfly (e.g. spatial transcriptomics or single-cell Hi-C), which the paper lists as future directions only
- When no model has been constructed/preprocessed — train_model requires load_data, data_preprocessing, and construct_model to have run first

## Inputs

- Preprocessed paired scRNA-seq and scATAC-seq AnnData matrices (from Butterfly.load_data + Butterfly.data_preprocessing)
- chrom_list: list of peak counts per chromosome for ATAC model construction
- Optional augmented training data from butterfly.augmentation (cell_type or MultiVI based)
- Pretrained per-modality autoencoder weights from the pretraining phase

## Outputs

- Trained scButterfly model checkpoint (e.g. trained_model.pt in './model')
- Trained translator and discriminator networks with dual-aligned latent representations
- Cross-modality predictions A2R_predict and R2A_predict via butterfly.test_model()

## How to apply

First run the two-stage schedule: pretrain each modality's autoencoder (RNA variational encoder with 50% input masking, ATAC with 30% masking) using modality reconstruction losses only, retaining those weights. Then call Butterfly.train_model to enter the integrative phase, jointly optimizing the translator (which samples from multivariate Gaussian latent representations) against two sigmoid-output discriminators per the paper's loss-weight schedule. Model construction must precede training via Butterfly.construct_model(chrom_list) with peaks sorted by chromosome so inter-chromosomal connections can be pruned for ATAC encoder/decoder. Discriminators output sigmoid activations and are trained adversarially against translated profiles produced by the translator. Keep default training parameters unless the documentation indicates otherwise, since the authors strongly recommend defaults for best results.

## Related tools

- **scButterfly** (Provides Butterfly.train_model, construct_model, and test_model implementing the adversarial translator–discriminator training within the dual-aligned VAE) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment required for the scButterfly package (conda create scButterfly python==3.9))
- **PyTorch** (Deep-learning backend for the encoders, translators, and sigmoid-output discriminators)

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id); butterfly.data_preprocessing(); butterfly.construct_model(chrom_list); butterfly.train_model(); A2R_predict, R2A_predict = butterfly.test_model()
```

## Evaluation signals

- trained_model.pt checkpoint exists in the output_path (default './model') after train_model completes
- butterfly.test_model runs end-to-end and returns both A2R_predict and R2A_predict matrices
- Translated profiles preserve cell-type structure, assessable with calculate_cluster_index and visualized via draw_tsne
- Discriminator outputs are sigmoid-bounded and adversarial loss converges per the specified loss-weight schedule
- ATAC decoder outputs pass through a Sigmoid layer, so predicted peak values fall in [0,1] before thresholding

## Limitations

- Unaugmented scButterfly-B training may underperform on noisy data from complex tissues; the paper notes data augmentation improves translation and alleviates limited multi-omics cell numbers
- Performance for cross-cell-type and cross-organ translation is flagged as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters
- Training on unpaired data requires the diagonal (augmentation-based) strategy rather than the standard paired adversarial schedule

## Evidence

- [abstract] Dual-aligned VAEs with adversarial training: "Here, we propose scButterfly, a versatile single-cell cross-modality translation method based on dual-aligned variational autoencoders and data augmentation schemes."
- [methods] Masked variational encoders and adversarial discriminators: [evidence span withheld — anchor status: not_found]
- [methods] train_model API call: "butterfly.Butterfly.train_model"
- [readme] Training invocation in README: "scButterfly model could be easily trained as following:
    
    python
    butterfly.train_model()
    "
- [readme] Test model produces cross-modality predictions: [evidence span withheld — anchor status: external_doc]
