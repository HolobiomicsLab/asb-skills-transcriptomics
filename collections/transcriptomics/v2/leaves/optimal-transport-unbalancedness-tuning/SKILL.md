---
name: optimal-transport-unbalancedness-tuning
description: Use when integrating unpaired single-cell modalities whose cell populations
  are unequally represented, and you need to verify that the chosen unbalanced-OT
  relaxation (transported mass m) yields robust, biologically meaningful embeddings.
when_to_use_negative:
- When prior information about the level of unbalanceness of the data is available
  — in that case m can be set directly rather than swept
- When modalities are fully paired or population proportions are balanced, where the
  unbalanced OT relaxation and its m parameter add unnecessary complexity
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scConfluence
  role: Fits uncoupled autoencoders with regularized Inverse Optimal Transport; refit
    at each m in the sensitivity grid; provides AutoEncoder, ScConfluence, fit(save_path='results')
    and get_latent()
  repo: https://github.com/cantinilab/scconfluence
- name: POT
  role: Computes the unbalanced OT plan via ot.partial.partial_wasserstein with the
    swept transported-mass parameter m
- name: Scanpy
  role: Evaluates the shared latent space via sc.pp.neighbors and sc.tl.umap on the
    'latent' representation for graph connectivity metrics
- name: Muon
  role: Handles the preprocessed unpaired MuData object and cross-modality annotations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:d56d6bfba0b9ca3684a34e07ebb06d90276c526b0bb676e140d4a7e779d795e8
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
  - POT
  - Scanpy
  - scConfluence
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# optimal-transport-unbalancedness-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sensitivity analysis of the unbalanced-OT transported-mass parameter m in scConfluence, sweeping m around the default 0.5 on progressively more unbalanced cell-line scenarios and scoring the shared latent space with cell-type purity, FOSCTTM, and graph connectivity.

## When to use

Use when integrating unpaired single-cell modalities whose cell populations are unequally represented, and you need to verify that the chosen unbalanced-OT relaxation (transported mass m) yields robust, biologically meaningful embeddings.

## When NOT to use

- When prior information about the level of unbalanceness of the data is available — in that case m can be set directly rather than swept
- When modalities are fully paired or population proportions are balanced, where the unbalanced OT relaxation and its m parameter add unnecessary complexity

## Inputs

- Preprocessed unpaired scCATseq MuData object with cross-modality cost matrix in mdata.uns['cross_modality1+modality2'] and cross_keys
- Cell-type labels for cells in each modality (for purity scoring)
- Progressively unbalanced scenario subsets (e.g. half K562 removed, all K562 removed, K562 and HCT removed)
- Grid of transported-mass values (e.g. 0.3, 0.4, 0.5, 0.6, 0.75)

## Outputs

- Latent embeddings per m value stored in mdata.obsm['latent']
- Metric-vs-m curves for cell-type purity, FOSCTTM, and graph connectivity
- sensitivity_curves.csv (tabulated metric values across m grid)
- sensitivity_fig13.png (sensitivity figure mirroring Supplementary Fig. 13)

## How to apply

Load the preprocessed unpaired MuData object (e.g. scCATseq cell-line scenarios with half K562 removed, all K562 removed, or K562+HCT removed) with the cross-modality cost matrix stored in mdata.uns (cross_modality1+modality2 with cross_keys). Build two scconfluence AutoEncoder objects, assemble them in a scConfluence model, and train at the default unbalanced-OT setting m=0.5 computed via ot.partial.partial_wasserstein. Then refit the model across a sensitivity grid (e.g. m = 0.3, 0.4, 0.5, 0.6, 0.75), extracting latent embeddings with model.get_latent() into mdata.obsm['latent'] at each value. Score each embedding with cell-type purity (kNN label agreement), FOSCTTM cross-modality nearest-neighbor consistency, and Scanpy graph connectivity (sc.pp.neighbors, sc.tl.umap on 'latent'). Plot metric-vs-m curves, compare against Supplementary Fig. 13, and write sensitivity_curves.csv and sensitivity_fig13.png.

## Related tools

- **scConfluence** (Fits uncoupled autoencoders with regularized Inverse Optimal Transport; refit at each m in the sensitivity grid; provides AutoEncoder, ScConfluence, fit(save_path='results') and get_latent()) — https://github.com/cantinilab/scconfluence
- **POT** (Computes the unbalanced OT plan via ot.partial.partial_wasserstein with the swept transported-mass parameter m)
- **Scanpy** (Evaluates the shared latent space via sc.pp.neighbors and sc.tl.umap on the 'latent' representation for graph connectivity metrics)
- **Muon** (Handles the preprocessed unpaired MuData object and cross-modality annotations)

## Examples

```
import scconfluence, ot; model = scconfluence.model.ScConfluence(mdata, unimodal_aes=autoencoders); model.fit(save_path='results'); mdata.obsm['latent'] = model.get_latent()  # sweep m in [0.3, 0.4, 0.5, 0.6, 0.75] via ot.partial.partial_wasserstein
```

## Evaluation signals

- Metric-vs-m curves for purity, FOSCTTM, and connectivity reproduce the qualitative pattern of Supplementary Fig. 13
- The default m=0.5 configuration falls in a flat, near-optimal region of all three metric curves (robust to m)
- Performance degradation across unbalancedness scenarios is milder for scConfluence than for benchmark methods (Seurat v3.0/v4.3.0, Liger, MultiMAP, Uniport, scGLUE)
- sensitivity_curves.csv contains one row per m value with all three metrics populated and finite
- Latent dimension is 16 with embeddings stored in mdata.obsm['latent'] after each refit

## Limitations

- The default m=0.5 is only justified when prior information about the level of unbalanceness is not available; strongly skewed data may require informed m
- Rare cell populations are hard to detect and under-represented in parameter estimation, especially with mini-batch gradient descent
- Supplementary Figs 13-18 (hyperparameter sensitivity) are referenced but not included in the main document, so exact reference curves must be obtained from the SI

## Evidence

- [discussion] We set by default m=0.5 which produces robust performances when prior information about the level of unbalanceness of the data is not available: [evidence span withheld — anchor status: not_found]
