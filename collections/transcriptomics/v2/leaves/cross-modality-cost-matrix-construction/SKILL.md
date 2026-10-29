---
name: cross-modality-cost-matrix-construction
description: Use when integrating unpaired single-cell modalities (e.g. scRNA-seq
  with scATAC-seq or surface protein counts) that share only a weak, partial set of
  features, and a cell-level coupling cost is needed for Optimal Transport-based alignment.
when_to_use_negative:
- Modalities share no feature connections at all (e.g. electrophysiology measurements
  vs gene expression), since scConfluence's alignment relies on cross-modality feature
  links
- Data are paired single-cell multiomic profiles of the same cells — paired integration
  does not require an OT coupling cost inferred from weak feature connections
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scConfluence
  role: Consumes cross_keys-linked feature cost matrices (Pearson similarity) as the
    coupling cost for unbalanced OT-based diagonal integration
  repo: https://github.com/cantinilab/scconfluence
- name: Muon
  role: Provides feature-level QC filtering (filter_var) and modality-specific normalization
    that precede cost matrix construction
- name: Scanpy
  role: Log-normalization of scRNA counts (normalize_total then log1p) feeding into
    the similarity computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:d0b5dc42393d0b1ab8e8bc6dcfcf866f895d5c9abc474502241609e8a29aad6c
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-modality-cost-matrix-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the cell-to-cell coupling cost between modalities from weakly connected features (e.g. genes shared between scRNA and scATAC peaks or surface proteins) using Pearson similarity, so scConfluence's unbalanced Optimal Transport aligns embeddings without converting all modalities to a common feature space.

## When to use

Use when integrating unpaired single-cell modalities (e.g. scRNA-seq with scATAC-seq or surface protein counts) that share only a weak, partial set of features, and a cell-level coupling cost is needed for Optimal Transport-based alignment.

## When NOT to use

- Modalities share no feature connections at all (e.g. electrophysiology measurements vs gene expression), since scConfluence's alignment relies on cross-modality feature links
- Data are paired single-cell multiomic profiles of the same cells — paired integration does not require an OT coupling cost inferred from weak feature connections

## Inputs

- Preprocessed MuData object with QC-filtered, normalized modalities (log-normalized scRNA, TF-IDF scATAC, CLR protein)
- Cross-modality feature connection/annotation linking features of different modalities (e.g. peaks to genes, antibody targets to genes), referenced by cross_keys in mdata.uns
- Normalized feature count matrices per modality

## Outputs

- Cross-modality cell-level distance/cost matrices from Pearson similarity on connected features, stored in mdata.uns under cross_keys
- Coupling cost matrix consumed by ot.partial.partial_wasserstein to compute the unbalanced OT plan

## How to apply

Load the preprocessed MuData object and, using the cross_keys entries stored in mdata.uns, compute cross-modality distance matrices from the connected features only — in scConfluence the cost matrix is derived with the Pearson similarity between features linked across modalities, avoiding lossy conversion of every modality to identical features. This converted feature cost matrix serves as the coupling cost between modalities for computing the unbalanced OT plan via ot.partial.partial_wasserstein with the default transported mass m=0.5. The resulting plan penalizes distances between plan-coupled embeddings in the 16-dimensional shared latent space, so the cost matrix directly shapes how modality encoders are back-propagated to align cells. Feature-level quality control (e.g. Muon's filter_var on genes or peaks) should precede cost construction so that noisy or rarely observed features do not dominate the similarity.

## Related tools

- **scConfluence** (Consumes cross_keys-linked feature cost matrices (Pearson similarity) as the coupling cost for unbalanced OT-based diagonal integration) — https://github.com/cantinilab/scconfluence
- **Muon** (Provides feature-level QC filtering (filter_var) and modality-specific normalization that precede cost matrix construction)
- **Scanpy** (Log-normalization of scRNA counts (normalize_total then log1p) feeding into the similarity computation)

## Examples

```
import scconfluence, mudata as md; mdata = md.read('data.h5mu'); model = scconfluence.model.AutoEncoder(mdata); model.fit(save_path='results'); mdata.obsm['latent'] = model.get_latent()
```

## Evaluation signals

- The cost matrix is computed only from connected features referenced by cross_keys, not from converted full-feature representations
- The unbalanced OT plan computed from this cost uses transported mass m=0.5 (ot.partial.partial_wasserstein default)
- Training on the resulting plan yields a mixed latent space without excessive loss of reconstruction quality; increasing sinkhorn_loss_weight or reach should increase mixing
- Per-modality biological structure is preserved in a UMAP of the aligned latent embeddings

## Limitations

- Converting all modalities to the same features causes loss of biological information because across-modality feature connections are rare and noisy
- scConfluence relies on cross-modality feature connections, which are not always available (e.g. electrophysiology vs gene expression)
- Rare cell populations are hard to detect and under-represented in parameter estimation, especially with mini-batch gradient descent

## Evidence

- [discussion] Default transported mass for unbalanced OT: [evidence span withheld — anchor status: not_found]
- [discussion] Coupling cost consumed by scConfluence training: [evidence span withheld — anchor status: not_found]
