---
name: h5ad-count-matrix-loading-and-cell-subsetting
description: Use when a raw single-cell count matrix (10X/h5ad/GSE accession) must
  be loaded into an anndata object and cells filtered by a compositional threshold
  (e.g. % human UMI counts) or zero-expression criteria before denoising or analysis.
when_to_use_negative:
- Input is already a filtered, normalized expression matrix — DCA requires raw counts
  without size-factor normalization, so re-filtering or normalizing would corrupt
  the denoising input
- The downstream task needs whole-population statistics across all sequenced cells
  rather than the species/quality subset defined by the compositional cutoff
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: anndata
  role: Loads and stores the h5ad-formatted HDF5 count matrix and metadata for filtering
    and subsetting
  repo: https://github.com/theislab/anndata
- name: Scanpy
  role: Python API compatible with the anndata object; provides filtering functions
    such as filter_genes_dispersion for HVG selection
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:c2e711df1cfea514714d1602c6f7b208aa010c2dd7e028e6e422c76ce865773a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/anndata
  tools:
  - DCA (deep count autoencoder network)
  - Keras
  - Scanpy
  - TensorFlow
  - anndata
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# h5ad count-matrix loading and cell subsetting

## Summary

Load single-cell count matrices into anndata/Scanpy (h5ad or 10X-style formats) and subset cells by quality/identity thresholds — e.g. reducing the Stoeckius et al. CITE-seq cord blood data to the 8,005 human cells by removing cells with less than 90% human UMI counts — to prepare a clean input matrix for downstream methods such as DCA denoising.

## When to use

Use when a raw single-cell count matrix (10X/h5ad/GSE accession) must be loaded into an anndata object and cells filtered by a compositional threshold (e.g. % human UMI counts) or zero-expression criteria before denoising or analysis.

## When NOT to use

- Input is already a filtered, normalized expression matrix — DCA requires raw counts without size-factor normalization, so re-filtering or normalizing would corrupt the denoising input
- The downstream task needs whole-population statistics across all sequenced cells rather than the species/quality subset defined by the compositional cutoff

## Inputs

- Single-cell gene expression count matrix (10X mtx, h5ad-formatted HDF5, or GEO accession e.g. GSE100866)
- Antibody-derived-tag (ADT) protein measurements from CITE-seq (optional, for paired protein-RNA data)
- Cell metadata/annotation used for filtering (e.g. per-cell human vs mouse UMI count fractions)

## Outputs

- anndata.AnnData object of raw counts for the retained cells (e.g. 8,005 human CITE-seq cells)
- Filtered cell/gene manifest recording which cells were removed by the 90% human UMI threshold

## How to apply

Read the count matrix with anndata/Scanpy (h5ad-formatted HDF5 or the raw counts from a GEO accession such as GSE100866), keeping raw integer counts and, where present, pairing the RNA layer with antibody-derived-tag (ADT) protein measurements. Apply the paper's filtering order: first remove cells and genes with zero expression from the count matrix, then apply dataset-specific compositional cutoffs — for the Stoeckius et al. CITE-seq cord blood data, remove cells with less than 90% human UMI counts to subset to the 8,005 human cells. Optionally restrict to highly variable genes when the downstream tool expects a reduced feature set (e.g. top 5000 or top 1000 HVGs via Scanpy's filter_genes_dispersion), but keep the full count matrix when feeding DCA, which runs on raw counts without size-factor normalization. Verify the resulting adata.n_obs matches the expected cell number and that the RNA cells align one-to-one with the ADT protein measurements before passing the object to downstream modeling.

## Related tools

- **anndata** (Loads and stores the h5ad-formatted HDF5 count matrix and metadata for filtering and subsetting) — https://github.com/theislab/anndata
- **Scanpy** (Python API compatible with the anndata object; provides filtering functions such as filter_genes_dispersion for HVG selection)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('cite_seq_cord_blood.h5ad'); adata = adata[adata.obs['pct_human_umi'] >= 0.90, :]; sc.pp.filter_cells(adata, min_counts=1); sc.pp.filter_genes(adata, min_counts=1)
```

## Evaluation signals

- adata.n_obs equals the expected subset size — e.g. exactly 8,005 human cells for the Stoeckius et al. cord blood data after the <90% human UMI removal
- No cells or genes with zero total expression remain in the filtered count matrix
- Count values remain raw integers (no size-factor normalization) so the object can be passed directly to DCA's negative binomial noise model
- For CITE-seq input, each retained RNA cell maps one-to-one to an ADT protein measurement row

## Limitations

- The 90% human UMI cutoff is dataset-specific (cord blood CITE-seq); other datasets need their own compositional thresholds
- Restricting to highly variable genes (e.g. top 1000 or 5000) discards genes needed for some analyses such as marker-level protein-mRNA correlation checks

## Evidence

- [discussion] data were subset to 8,005 human cells by removing cells with less than 90% human UMI counts.: "data were subset to 8,005 human cells by removing cells with less than 90% human UMI counts."
- [discussion] it supports h5ad-formatted HDF5 files: "it supports h5ad-formatted HDF5 files (https://github.com/theislab/anndata)"
- [discussion] First, cells and genes with zero expression are removed from the count matrix.: "First, cells and genes with zero expression are removed from the count matrix."
- [discussion] the top 1000 highly variable genes are selected using “filter_genes_dispersion” function of Scanpy: "the top 1000 highly variable genes are selected using “filter_genes_dispersion” function of Scanpy"
