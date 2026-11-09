---
name: scrna-seq-preprocessing-scanpy
description: Use when a raw or minimally processed scRNA-seq count matrix (e.g. Veres
  et al. stage-5 beta-like differentiation, 25,299 cells) must be turned into a cell-type-annotated
  adata object before running FateCompass.
when_to_use_negative:
- The adata object is already pre-processed and cell-type annotated (e.g. the Veres
  et al. stage-5 object shipped with the FateCompass reproducibility scripts) — load
  it directly instead of redoing pre-processing
- The data is bulk RNA-seq or another non-single-cell modality, where the article's
  framing (ISMARA/DOROTHEA for bulk, FateCompass for scRNA-seq) does not apply
- You only need downstream steps (trajectory building, tf_activities.m, bootstrapping.m,
  differential_motif_activity_analysis.m) and no gene-space alignment with the binding-site
  matrix is required
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Scanpy
  role: Perform basic pre-processing (normalization, filtering, annotation) of the
    single-cell data into the adata object consumed by FateCompass
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scVelo
  role: Optionally compute the RNA velocity field stored in the adata, used by FateCompass
    when RNA-velocity-driven trajectories are chosen over differentiation potential
  repo: https://scvelo.readthedocs.io/en/stable/
- name: FateCompass
  role: Downstream consumer of the pre-processed adata; its Python module fatecompass.py
    requires the adata plus the TF binding-site matrix
  repo: github.com/sarajimenez/fatecompass
- name: dropEst
  role: Upstream pipeline that can produce the droplet-based count matrices fed into
    Scanpy pre-processing
  repo: https://dropest.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:33318877849b631b2c9f6b9825c2158e8e428bb79fd5299ab7f4482844463349
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  tools:
  - FateCompass
  - Python
  - Scanpy
  - dropEst
  - scVelo
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# scRNA-seq-preprocessing-scanpy

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare a single-cell RNA-seq dataset as a pre-processed AnnData object (cell-type annotated, optionally with an RNA velocity field) using Scanpy and scVelo, so it can serve as input to FateCompass trajectories and TF activity inference.

## When to use

Use when a raw or minimally processed scRNA-seq count matrix (e.g. Veres et al. stage-5 beta-like differentiation, 25,299 cells) must be turned into a cell-type-annotated adata object before running FateCompass.

## When NOT to use

- The adata object is already pre-processed and cell-type annotated (e.g. the Veres et al. stage-5 object shipped with the FateCompass reproducibility scripts) — load it directly instead of redoing pre-processing
- The data is bulk RNA-seq or another non-single-cell modality, where the article's framing (ISMARA/DOROTHEA for bulk, FateCompass for scRNA-seq) does not apply
- You only need downstream steps (trajectory building, tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m) and no gene-space alignment with the binding-site matrix is required

## Inputs

- Raw or minimally processed single-cell RNA-seq count matrix (e.g. from dropEst) importable as AnnData/adata
- Cell-type (cluster) annotations for the cells
- Optional: spliced/unspliced counts for scVelo RNA velocity estimation
- TF binding-site matrix (rows = expressed genes, columns = TFs/motifs), e.g. from the FateCompass Veres stage-5 dataset

## Outputs

- Pre-processed AnnData (adata) object with 'clusters' cell-type annotations (e.g. 25,299 Veres stage-5 cells) and optionally an RNA velocity field
- Environment configured via fatecompass.yml for running the fatecompass.py module

## How to apply

Run basic pre-processing (filtering, normalization, and feature handling) with Scanpy, and compute the RNA velocity field with scVelo if velocity-driven trajectories are planned. Store the result as an adata object annotated with at least cell types in the 'clusters' column, since FateCompass requires (i) a pre-processed adata with cell-type annotations and (ii) a TF binding-site matrix whose rows are the expressed genes and columns the TFs/motifs. Set up the environment with the provided fatecompass.yml conda file so all Python dependencies of the fatecompass.py module are available. For the Veres stage-5 reproduction, load the pre-processed 25,299-cell cell-type-annotated adata object directly rather than re-deriving it. Rationale: FateCompass's Markov-chain trajectories and the linear model of gene regulation consume the adata and binding-site matrix jointly, so gene space consistency between the two inputs is essential.

## Related tools

- **Scanpy** (Perform basic pre-processing (normalization, filtering, annotation) of the single-cell data into the adata object consumed by FateCompass) — https://scanpy.readthedocs.io/en/stable/
- **scVelo** (Optionally compute the RNA velocity field stored in the adata, used by FateCompass when RNA-velocity-driven trajectories are chosen over differentiation potential) — https://scvelo.readthedocs.io/en/stable/
- **FateCompass** (Downstream consumer of the pre-processed adata; its Python module fatecompass.py requires the adata plus the TF binding-site matrix) — github.com/sarajimenez/fatecompass
- **dropEst** (Upstream pipeline that can produce the droplet-based count matrices fed into Scanpy pre-processing) — https://dropest.readthedocs.io/en/latest/

## Examples

```
conda env create -f fatecompass.yml; conda install notebook; jupyter notebook tutorials/fatecompass_basics_differentiation_potential.ipynb
```

## Evaluation signals

- The resulting adata carries a 'clusters' column with cell-type annotations, the minimum annotation FateCompass requires
- Rows of the binding-site matrix (expressed genes) match the genes retained in the pre-processed adata, since the linear model of gene regulation operates on this alignment
- For the Veres et al. stage-5 reproduction, the object contains 25,299 annotated cells as in the reference workflow
- The conda environment created from fatecompass.yml can import the fatecompass.py module without dependency errors
- If velocity-driven trajectories are planned, the adata includes a velocity field computed with scVelo (La Manno et al. 2018 / Bergen et al. 2020 implementations)

## Limitations

- FateCompass depends on input parameters that must be hand tuned; detailed tutorials are provided but automatic parameter selection is not available
- The binding-site matrix relies on computationally predicted regulatory sites (promoter regions only), introducing a bias in the gene-regulatory network structure downstream of pre-processing
- Pre-processing choices should avoid over-reliance on UMAP or similar dimensionality reductions, which the authors note lack biological interpretability for trajectory building on the nearest-neighbor graph

## Evidence

- [full_text] Load the pre-processed Veres et al. stage-5 adata object (cell-type annotated, 25,299 cells) and the TF binding-site matrix: [evidence span withheld — anchor status: not_found]
