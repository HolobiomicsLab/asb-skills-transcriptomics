---
name: neuronal-morphology-scrna-integration
description: Use when integrating unpaired morphology image features with scRNA-seq
  profiles (e.g. Patch-seq 625 morphologies with 589 RNA-only cells) to test whether
  scRNA-defined populations, like IT neurons, show molecular-morphological structure.
when_to_use_negative:
- No cross-modality feature connections are available for the modality pair — scConfluence
  relies on connections between features of different modalities, e.g. it is not applicable
  when integrating electrophysiology measurements with gene expression
- The goal requires integrating paired (same-cell) multimodal data jointly with unpaired
  data — this paired/unpaired combination is not yet addressed by scConfluence
- The analysis targets rare cell populations that are the primary object of study
  — rare populations are hard to detect and under-represented in mini-batch parameter
  estimation
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2229
tools:
- name: scConfluence
  role: 'Core diagonal-integration method: uncoupled per-modality AutoEncoders plus
    regularized Inverse Optimal Transport on the cross-modality cost matrix yield
    the shared latent embedding for morphologies and scRNA-seq'
  repo: https://github.com/cantinilab/scconfluence
- name: Scanpy
  role: Neighbor graph computation (sc.pp.neighbors on use_rep='latent') and UMAP
    visualization colored by tufted/untufted labels and Fezf2 expression
- name: Muon
  role: Provides the MuData container in which morphology and scRNA modalities, mdata.uns
    cross-modality distances, and latent obsm entries are stored
- name: scipy (cdist)
  role: Computes the cross-modality distance matrix on weakly connected morphological/gene
    features, stored in mdata.uns with the cross_keys entry
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:2acae50e770cbe19598878ea387b154792d98467e65fe7ee07f6aed3cc7a2d7e
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
  - scipy (cdist)
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# neuronal-morphology-scrna-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Diagonal integration of neuronal morphology images with scRNA-seq using scConfluence's uncoupled autoencoders and regularized Inverse Optimal Transport on weakly connected features, applied to mouse primary motor cortex Patch-seq data to reveal molecular-morphological associations such as the Fezf2/apical-dendrite gradient in Intra Telencephalic (IT) neurons.

## When to use

Use when integrating unpaired morphology image features with scRNA-seq profiles (e.g. Patch-seq 625 morphologies with 589 RNA-only cells) to test whether scRNA-defined populations, like IT neurons, show molecular-morphological structure.

## When NOT to use

- No cross-modality feature connections are available for the modality pair — scConfluence relies on connections between features of different modalities, e.g. it is not applicable when integrating electrophysiology measurements with gene expression
- The goal requires integrating paired (same-cell) multimodal data jointly with unpaired data — this paired/unpaired combination is not yet addressed by scConfluence
- The analysis targets rare cell populations that are the primary object of study — rare populations are hard to detect and under-represented in mini-batch parameter estimation

## Inputs

- MuData object with 625 neuronal morphology profiles and 589 scRNA-only Patch-seq cells from the mouse primary motor cortex
- Cross-modality feature connections (weakly connected morphological/gene features) linking morphology measurements to genes
- Cell-type/annotation metadata (IT neuron tufted/untufted labels) and gene expression values (e.g. Fezf2)

## Outputs

- Shared 16-dimensional latent embedding in mdata.obsm['latent'] spanning both modalities
- UMAP embedding colored by tufted/untufted IT neuron labels and Fezf2 expression (Fig. 6b-d analog)
- Demonstrated molecular-morphological association: higher Fezf2 in tufted layer-5 IT neurons with a joint Fezf2/apical-dendrite gradient

## How to apply

Load the MuData object holding the 625 morphology and 589 scRNA-only Patch-seq cells, and compute the cross-modality distance matrix on weakly connected morphological/gene features with scipy cdist, storing it in mdata.uns under the cross_keys entry. Initialize one scconfluence.model.AutoEncoder per modality, then assemble them into a scconfluence.model.ScConfluence model and train with .fit(save_path='results') using the default latent dimension of 16 and unbalanced Sinkhorn regularization with transported mass m=0.5. Extract the shared latent embeddings with model.get_latent(), write them into mdata.obsm['latent'] aligned to mdata.obs_names, then build a neighbor graph with sc.pp.neighbors(mdata, use_rep='latent') and embed with sc.tl.umap(mdata). The rationale is that weakly connected features across modalities (rare and noisy by nature) suffice for rIOT to align cells, avoiding lossy feature conversion. Validate by coloring the UMAP by tufted/untufted IT neuron labels and Fezf2 expression to reproduce Fig. 6b-d and confirming with expert review that tufted and untufted neurons organize along the apical-dendrite manifold with graded Fezf2 expression.

## Related tools

- **scConfluence** (Core diagonal-integration method: uncoupled per-modality AutoEncoders plus regularized Inverse Optimal Transport on the cross-modality cost matrix yield the shared latent embedding for morphologies and scRNA-seq) — https://github.com/cantinilab/scconfluence
- **Scanpy** (Neighbor graph computation (sc.pp.neighbors on use_rep='latent') and UMAP visualization colored by tufted/untufted labels and Fezf2 expression)
- **Muon** (Provides the MuData container in which morphology and scRNA modalities, mdata.uns cross-modality distances, and latent obsm entries are stored)
- **scipy (cdist)** (Computes the cross-modality distance matrix on weakly connected morphological/gene features, stored in mdata.uns with the cross_keys entry)

## Examples

```
import scconfluence; model = scconfluence.model.ScConfluence(ae_dict={'morpho': ae_morpho, 'rna': ae_rna}); model.fit(save_path='results'); mdata.obsm['latent'] = model.get_latent(); sc.pp.neighbors(mdata, use_rep='latent'); sc.tl.umap(mdata)
```

## Evaluation signals

- The cross-modality cost matrix is present in mdata.uns under the cross_keys entry with shape (625 morphologies, 589 RNA cells)
- mdata.obsm['latent'] contains embeddings for all 1214 cells (625 morphology + 589 scRNA) and row names align exactly with mdata.obs_names
- UMAP colored by tufted/untufted IT neuron labels shows tufted and untufted neurons organized along the apical-dendrite manifold (Fig. 6b-d pattern)
- UMAP colored by Fezf2 expression shows a graded pattern jointly varying with the apical-dendrite gradient, with higher Fezf2 in tufted layer-5 IT neurons
- Expert review confirms the tufted-to-untufted progression of IT neurons along the latent manifold

## Limitations

- Rare cell populations are much less likely to be simultaneously sampled from each modality in mini-batches, though mini-batch optimization is necessary to scale to millions of cells
- scConfluence relies on connections between features of different modalities, which are not always available (e.g. electrophysiology vs gene expression)
- Hidden-layer sizes were adapted per dataset and documented only in Supplementary Table 5, which is not provided, complicating exact reproduction

## Evidence

- [methods] We set by default m=0.5 which produces robust performances when prior information about the level of unbalanceness of the data is not available: [evidence span withheld — anchor status: not_found]
