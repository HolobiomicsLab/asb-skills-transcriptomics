---
name: masked-input-encoder-construction
description: Use when constructing the basic scButterfly-B encoder/decoder stack for
  paired scRNA-seq and scATAC-seq translation and dropout noise must be denoised without
  data augmentation.
when_to_use_negative:
- The goal is scButterfly-T or scButterfly-C, which rely on cell-type or MultiVI data
  augmentation rather than the basic masked-encoder architecture (aug_type must be
  None for scButterfly-B).
- Training on unpaired data with diagonal strategies, which requires the scButterfly
  extension for unpaired training rather than the standard paired-input masked encoders.
- Modalities other than RNA/ATAC/ADT single-cell count matrices, e.g., spatial transcriptomics
  or single-cell Hi-C, which are named as future directions not yet supported.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scButterfly
  role: Provides the Butterfly API (load_data, data_preprocessing, construct_model,
    train_model, test_model) that builds and trains the masked dual-aligned VAE on
    raw count matrices
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment in which scButterfly is installed via pip or cloned from
    GitHub
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:236b081b684e4be4ad285951a0558b08809979bf13764a9b3841956de8902c33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - PyTorch
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# masked-input-encoder-construction

## Summary

Build the fixed seven-module scButterfly-B network whose variational encoders apply modality-specific random input masking (50% for RNA, 30% for ATAC) to alleviate dropout-driven false zeros while keeping the model parameter-efficient via pruned inter-chromosomal ATAC connections.

## When to use

Use when constructing the basic scButterfly-B encoder/decoder stack for paired scRNA-seq and scATAC-seq translation and dropout noise must be denoised without data augmentation.

## When NOT to use

- The goal is scButterfly-T or scButterfly-C, which rely on cell-type or MultiVI data augmentation rather than the basic masked-encoder architecture (aug_type must be None for scButterfly-B).
- Training on unpaired data with diagonal strategies, which requires the scButterfly extension for unpaired training rather than the standard paired-input masked encoders.
- Modalities other than RNA/ATAC/ADT single-cell count matrices, e.g., spatial transcriptomics or single-cell Hi-C, which are named as future directions not yet supported.

## Inputs

- Raw count matrices as AnnData objects: RNA_data (n_obs x n_vars, genes) and ATAC_data (n_obs x n_vars, peaks)
- Lists of cell IDs: train_id, test_id, optional validation_id
- chrom_list: list of peak counts for each chromosome, peaks sorted by chromosome

## Outputs

- Constructed scButterfly-B PyTorch model with masked variational encoders, translator, discriminators, and decoders
- Trained model checkpoint (e.g., trained_model.pt in ./model output path)
- Cross-modality translated profiles from test_model (A2R_predict, R2A_predict)

## How to apply

Instantiate a Butterfly object and call load_data with the raw scRNA-seq AnnData (cells x genes) and scATAC-seq AnnData (cells x peaks) plus train/test/validation IDs, then run data_preprocessing which applies RNA normalization and ATAC peak handling including intra-chromosomal connection pruning. Call construct_model(chrom_list) — where chrom_list is the list of peak counts per chromosome and peaks must be sorted by chromosome — to build the fixed seven-module network: RNA and ATAC variational encoders with 50% (RNA) and 30% (ATAC) input masking, a translator sampling from multivariate Gaussian latent representations, two sigmoid-output discriminators, and decoders with a Sigmoid output layer for the ATAC modality. The masking strategy addresses dropout noise, while pruning inter-chromosomal connections in the ATAC encoder/decoder keeps the model parameter-efficient. Pretrain the per-modality autoencoders with reconstruction losses only, then run train_model to jointly optimize translator and discriminator adversarial losses, retaining pretraining weights per the loss-weight schedule. Masking rates are the key thresholds: keep the defaults (RNA 50%, ATAC 30%) unless the documentation advises otherwise.

## Related tools

- **scButterfly** (Provides the Butterfly API (load_data, data_preprocessing, construct_model, train_model, test_model) that builds and trains the masked dual-aligned VAE on raw count matrices) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment in which scButterfly is installed via pip or cloned from GitHub)

## Examples

```
from scButterfly.butterfly import Butterfly; b = Butterfly(); b.load_data(RNA_data, ATAC_data, train_id, test_id); b.data_preprocessing(); b.construct_model(chrom_list); b.train_model()
```

## Evaluation signals

- Model structure logging from construct_model shows variational encoders with the specified masking (RNA 50%, ATAC 30%), two sigmoid-output discriminators, and a Sigmoid output layer on the ATAC decoder.
- trained_model.pt checkpoint exists in the configured output_path after train_model completes.
- Butterfly.test_model runs end-to-end on held-out single-modality inputs, returning both A2R_predict and R2A_predict.
- Translated profiles preserve cell-type structure, assessable with calculate_cluster_index and visualizable via draw_tsne.
- ATAC peak matrix was preprocessed with intra-chromosomal connection pruning and peaks were sorted by chromosome before constructing the model, matching chrom_list.

## Limitations

- Basic scButterfly-B does not use data augmentation; better predictions require scButterfly-T (cell_type_augmentation) or scButterfly-C (MultiVI_augmentation) at extra training cost.
- Performance for cross-cell-type and cross-organ translation is flagged as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters.
- Spatial transcriptomics and single-cell Hi-C are future extensions not currently supported.

## Evidence

- [intro] dropout noise: "dropout events due to the loss of DNA material during library preparation require tailored denoising approaches for the large number of false zeros"
- [readme] construct_model usage: [evidence span withheld — anchor status: external_doc]
- [readme] load_data contract: [evidence span withheld — anchor status: external_doc]
- [intro] masked VAE design: "we developed scButterfly, a versatile framework capable of single-cell cross-modality translation and multiple extensive applications"
