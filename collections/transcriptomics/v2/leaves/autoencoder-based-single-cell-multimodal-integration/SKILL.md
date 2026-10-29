---
name: autoencoder-based-single-cell-multimodal-integration
description: Use when integrating unpaired (non-co-assayed) single-cell modalities
  such as scRNA-seq with scATAC-seq, surface protein, CyTOF, smFISH, or morphology
  into a shared low-dimensional latent space for visualization, clustering, and cross-modality
  imputation.
when_to_use_negative:
- Modalities lack cross-modality feature connections entirely (e.g. electrophysiology
  measurements vs gene expression), since the rIOT alignment relies on connections
  between features of different modalities
- Data are paired/co-assayed multimodal, or a mix of paired and unpaired multimodal
  data must be jointly integrated — scConfluence addresses diagonal integration of
  unpaired data only
- 'The goal requires handling rare cell populations reliably: rare populations are
  unlikely to be simultaneously sampled from each modality in mini-batches, a stated
  challenging limitation'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scConfluence
  role: 'Core method: provides scconfluence.model.AutoEncoder (uncoupled encoders
    per modality), model.fit for rIOT-aligned training with unbalanced OT plan and
    Sinkhorn divergence regularization, and model.get_latent for embedding extraction'
  repo: https://github.com/cantinilab/scconfluence
- name: Scanpy
  role: 'scRNA preprocessing within the scverse workflow: normalize_total to 10,000
    counts and log1p transformation'
- name: Muon
  role: 'MuData container handling plus modality-specific normalization: tfidf for
    scATAC, Center Log Ratio for protein, filter_obs/filter_var for QC filtering'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:66e50844b2786ad117b1ea45217f65f93c7690f1552cd48573f47fbf0b627bfe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cantinilab/scconfluence
  repo_url: https://github.com/cantinilab/scconfluence
  tools:
  - Muon
  - Scanpy
  - scConfluence
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# autoencoder-based-single-cell-multimodal-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Diagonal integration of unpaired single-cell multiomics data by training one uncoupled AutoEncoder per modality to map features into a shared 16-dimensional latent space, aligned with a regularized Inverse Optimal Transport (rIOT) objective computed from weakly connected cross-modality features.

## When to use

Use when integrating unpaired (non-co-assayed) single-cell modalities such as scRNA-seq with scATAC-seq, surface protein, CyTOF, smFISH, or morphology into a shared low-dimensional latent space for visualization, clustering, and cross-modality imputation.

## When NOT to use

- Modalities lack cross-modality feature connections entirely (e.g. electrophysiology measurements vs gene expression), since the rIOT alignment relies on connections between features of different modalities
- Data are paired/co-assayed multimodal, or a mix of paired and unpaired multimodal data must be jointly integrated — scConfluence addresses diagonal integration of unpaired data only
- The goal requires handling rare cell populations reliably: rare populations are unlikely to be simultaneously sampled from each modality in mini-batches, a stated challenging limitation

## Inputs

- Preprocessed MuData object with per-modality AnnData layers (scRNA log-normalized to 10,000 counts + log1p, scATAC TF-IDF normalized, protein Center Log Ratio normalized)
- Cross-modality cost/distance matrices computed from weakly connected features, stored in mdata.uns keyed by cross_keys
- Batch index annotation per cell (decoder covariate)
- Optional: first 100 principal components of log-normalized counts as encoder input

## Outputs

- mdata.obsm['latent'] — mixed, modality-aligned cell embeddings in a shared 16-dimensional latent space
- latent_embeddings.npy — saved embedding matrix for downstream visualization, clustering, and cross-modality feature imputation
- Trained uncoupled AutoEncoders (one per modality) saved under the fit save_path (e.g. 'results')

## How to apply

Load a preprocessed MuData object and use cross-modality distance matrices computed from connected features (stored in mdata.uns with cross_keys) as the coupling cost; scConfluence computes this cost with Pearson similarity on common feature matrices. Initialize one uncoupled scconfluence.model.AutoEncoder per modality mapping into a shared 16-dimensional latent space, using the first 100 principal components as encoder input and the batch index as a decoder covariate for batch correction. Compute the unbalanced OT plan between latent embeddings of each modality pair via ot.partial.partial_wasserstein with default transported mass m=0.5, which produces robust performance when the level of unbalancedness is unknown. Train with model.fit(save_path='results'), minimizing the IOT distance term penalizing plan-coupled embedding distances plus the unbalanced Sinkhorn divergence regularizer weighted by sinkhorn_loss_weight, with reach controlling the discarded mass; gradients are back-propagated through both modality encoders. Extract aligned embeddings with model.get_latent() into mdata.obsm['latent']. Modality-specific preprocessing matters: scRNA gets log-normalization to 10,000 counts plus log1p, scATAC gets TF-IDF (Muon), and protein gets Center Log Ratio normalization (Muon).

## Related tools

- **scConfluence** (Core method: provides scconfluence.model.AutoEncoder (uncoupled encoders per modality), model.fit for rIOT-aligned training with unbalanced OT plan and Sinkhorn divergence regularization, and model.get_latent for embedding extraction) — https://github.com/cantinilab/scconfluence
- **Scanpy** (scRNA preprocessing within the scverse workflow: normalize_total to 10,000 counts and log1p transformation)
- **Muon** (MuData container handling plus modality-specific normalization: tfidf for scATAC, Center Log Ratio for protein, filter_obs/filter_var for QC filtering)

## Examples

```
import scconfluence, mudata as md; mdata = md.read('pbmc_multiome.h5mu'); model = scconfluence.model.AutoEncoder(mdata); model.fit(save_path='results'); mdata.obsm['latent'] = model.get_latent()
```

## Evaluation signals

- The fitted objective yields a mixed latent space: increasing sinkhorn_loss_weight or reach should increase mixing of modalities in the embedding
- Reconstruction quality from the decoders remains acceptable (not excessively degraded by the alignment loss)
- Per-modality biological structure is preserved in a UMAP of mdata.obsm['latent'] (e.g. B cells and Monocytes remain distinguishable)
- Latent dimension equals 16 for all datasets, matching the paper's configuration
- The unbalanced OT plan uses default transported mass m=0.5 via ot.partial.partial_wasserstein, allowing mass to be discarded so modality-specific cell types do not force spurious couplings

## Limitations

- Rare cell populations are hard to detect and under-represented in parameter estimation, especially with mini-batch gradient descent, which is nevertheless required to scale to millions of cells
- scConfluence relies on cross-modality feature connections, which are not always available (e.g. electrophysiology vs gene expression)
- Integration of paired with unpaired multimodal data is not yet addressed by scConfluence
- Latent dimension is 16 but hidden layer sizes must be adapted to dataset dimensionality (Supplementary Table 5), which is not provided in the main document

## Evidence

- [abstract] rIOT alignment objective: "scConfluence combines uncoupled autoencoders on the complete set of features with regularized Inverse Optimal Transport on weakly connected features."
- [discussion] unbalanced OT plan: [evidence span withheld — anchor status: not_found]
- [discussion] cost matrix from connected features: "we use the Pearson similarity (see Implementation details) to compute the cost matrix"
- [discussion] latent dimension: "We used a latent dimension of 16 for all datasets but adapted the number of neurons in hidden layers to the dimensionality of the datasets"
- [readme] package interface: "scConfluence takes as an input a MuData object and populates its `obsm` field with the latent embeddings."
- [intro] unbalanced relaxation benefit: "thanks to the unbalanced relaxation of Optimal Transport33, scConfluence can also deal with cell types absent in a modality"
