---
name: cross-modality-marker-enrichment-testing
description: Use when after multi-omic integration you have reclustered cells (e.g.
  B cells and monocytes at Louvain resolution 0.2) and need statistical evidence that
  subcluster markers are shared across modalities (scRNA vs scATAC) rather than an
  alignment artifact.
when_to_use_negative:
- Modalities lack cross-modality feature connections (e.g. integrating electrophysiology
  measurements with gene expression), so scConfluence embeddings and hence cross-modality
  marker overlap cannot be computed
- Cell populations of interest are extremely rare (<0.5% of the dataset); the article
  notes rare populations are hard to detect and under-represented in parameter estimation,
  undermining marker reliability
- Input is paired multimodal data on the same cells, where joint-feature methods rather
  than diagonal integration plus cross-modality overlap testing are appropriate
edam_operation: http://edamontology.org/operation_0315
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scConfluence
  role: Produces the shared latent embeddings (uncoupled autoencoders + regularized
    Inverse Optimal Transport) on which clustering and subclustering precede marker
    overlap testing
  repo: https://github.com/cantinilab/scconfluence
- name: Muon
  role: Loads the tri-omics PBMC MuData object (md.read_h5mu) and provides per-modality
    normalization (tfidf for scATAC, CLR for protein, filter_obs/filter_var QC)
- name: Scanpy
  role: Builds the neighbor graph on 'latent' (sc.pp.neighbors), clusters with Louvain
    (resolutions 0.5 and 0.2), computes UMAP, and derives cluster marker genes
- name: SciPy
  role: supplies cdist for cross-modality distance matrices in mdata.uns['cross_keys']
    and Fisher's exact tests on cross-modality marker overlaps
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:83c59018289b09b4e2f3167d70dd58ef702e60385e7ec6b3792c0453ea6e145f
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
  - SciPy
  - scConfluence
  edam_operation: http://edamontology.org/operation_0315
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-modality-marker-enrichment-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates diagonal integration of unpaired single-cell modalities by testing whether differentially expressed genes of subclusters (e.g. re-annotated B cell and monocyte subclusters from tri-omics PBMC scRNA/scATAC/CyTOF data) significantly overlap across modalities using Fisher's exact tests, confirming biological signal rather than artificial alignment.

## When to use

Use when after multi-omic integration you have reclustered cells (e.g. B cells and monocytes at Louvain resolution 0.2) and need statistical evidence that subcluster markers are shared across modalities (scRNA vs scATAC) rather than an alignment artifact.

## When NOT to use

- Modalities lack cross-modality feature connections (e.g. integrating electrophysiology measurements with gene expression), so scConfluence embeddings and hence cross-modality marker overlap cannot be computed
- Cell populations of interest are extremely rare (<0.5% of the dataset); the article notes rare populations are hard to detect and under-represented in parameter estimation, undermining marker reliability
- Input is paired multimodal data on the same cells, where joint-feature methods rather than diagonal integration plus cross-modality overlap testing are appropriate

## Inputs

- Preprocessed tri-omics PBMC MuData object (scRNA-seq, scATAC-seq, CyTOF) with mdata.uns['cross_keys'] holding cross-modality distance matrices (scipy cdist)
- scConfluence shared latent embeddings in mdata.obsm['latent'] aligned to mdata.obs_names
- Louvain cluster and modality labels on the latent UMAP (coarse resolution 0.5; B cell/monocyte subsets reclusters at resolution 0.2)
- Per-modality marker gene lists for each B cell / monocyte subcluster

## Outputs

- marker_overlap_fisher_tests.csv with cross-modality marker overlap Fisher's exact test results and significance fields
- Validated B cell subcluster annotations (memory, naive, plasma) and monocyte subclusters with counts of overlapping DE genes per cluster (e.g. 30/12/232 for clusters 0/1/2)
- UMAP of inferred embeddings colored by cluster and modality showing preserved B cell and monocyte structure

## How to apply

After fitting scConfluence on the tri-omics PBMC MuData object (uncoupled autoencoders per modality plus rIOT on weakly connected features), extract shared embeddings via model.get_latent() into mdata.obsm['latent'], build a neighbor graph with sc.pp.neighbors(use_rep='latent'), and cluster with Louvain at resolution 0.5 before subsetting B cells and monocytes and reclustering at resolution 0.2. Identify marker genes for each subcluster independently in each modality, then run Fisher's exact tests on the overlap of cross-modality markers between B cell and monocyte subclusters. Write the test table (with significance fields) to marker_overlap_fisher_tests.csv. Rationale: significant overlap of differentially expressed genes between scRNA and scATAC subclusters — e.g. 30 genes for cluster 0, 12 for cluster 1, 232 for cluster 2 — demonstrates the integration re-annotated the scRNA B cell cluster into memory, naive and plasma subclusters with real, shared biological signal.

## Related tools

- **scConfluence** (Produces the shared latent embeddings (uncoupled autoencoders + regularized Inverse Optimal Transport) on which clustering and subclustering precede marker overlap testing) — https://github.com/cantinilab/scconfluence
- **Muon** (Loads the tri-omics PBMC MuData object (md.read_h5mu) and provides per-modality normalization (tfidf for scATAC, CLR for protein, filter_obs/filter_var QC))
- **Scanpy** (Builds the neighbor graph on 'latent' (sc.pp.neighbors), clusters with Louvain (resolutions 0.5 and 0.2), computes UMAP, and derives cluster marker genes)
- **SciPy** (supplies cdist for cross-modality distance matrices in mdata.uns['cross_keys'] and Fisher's exact tests on cross-modality marker overlaps)

## Examples

```
import scanpy as sc; from scipy.stats import fisher_exact; sc.pp.neighbors(mdata, use_rep='latent'); sc.tl.louvain(mdata, resolution=0.5); odds, p = fisher_exact(rna_markers & atac_markers, universe)
```

## Evaluation signals

- marker_overlap_fisher_tests.csv exists and contains significance fields (p-values) for the B cell and monocyte cross-modality marker overlap tests
- Differentially expressed genes of B cell subclusters significantly overlap between scRNA and scATAC: 30 genes for cluster 0, 12 for cluster 1, 232 for cluster 2
- B cell cluster is re-annotated into biologically coherent memory, naive and plasma subclusters supported by shared markers across modalities
- UMAP of the inferred embeddings preserves B cell and monocyte structure when colored by cluster and modality labels
- mdata.obsm['latent'] row indices align to mdata.obs_names before neighbor graph computation

## Limitations

- Rare cell populations are hard to detect and under-represented in parameter estimation, especially with mini-batch gradient descent, which can destabilize subcluster markers
- scConfluence relies on cross-modality feature connections, which are unavailable for some modality pairs (e.g. electrophysiology vs gene expression), blocking this validation strategy
- Validation relies on significant marker overlap being real biological signal; small overlap counts (e.g. 12 genes) should be interpreted cautiously

## Evidence

- [discussion] Fisher's exact tests on cross-modality marker overlaps: [evidence span withheld — anchor status: not_found]
