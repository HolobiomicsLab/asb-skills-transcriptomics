---
name: single-cell-rnaseq-preprocessing-filtering
description: Use when raw scRNA-seq count data (e.g., GEO datasets like GSE117872)
  must be cleaned prior to bulk-to-single-cell drug response prediction, and cells/genes
  failing QC thresholds must be removed without discarding heterogeneity.
when_to_use_negative:
- Input is already a preprocessed, QC-filtered matrix or a trained-model output such
  as an AnnData .h5ad with sens_label — re-filtering risks removing signal
- The analysis target is bulk RNA-seq drug response data (GDSC/CCLE cell-line matrices),
  which scDEAL consumes directly in bulkmodel.py without single-cell QC filters
- Non-human scRNA-seq data requiring homolog-gene mapping first (e.g., the mouse GSE110894
  case where only homologous genes were kept before the human-trained model was applied)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: Provides the AnnData-based filtering/preprocessing utilities used to apply
    min_g/min_c/percent_mito and highly-variable-gene dispersion filters
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: AnnData
  role: Container format for filtered single-cell data and for the final output .h5ad
    with sens_label in adata.obs
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: scDEAL
  role: Framework whose scanpypip utilities expose the filtering parameters (min_n_genes,
    max_n_genes, min_g, min_c, percent_mito, var_genes_disp) applied before model
    training
  repo: https://github.com/OSU-BMBL/scDEAL.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:21102051fad06e5ddb09c19820f88683396e4dd0ce24693198c61e925fcb430e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - Scanpy
  - conda-pack
  - scDEAL
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-rnaseq-preprocessing-filtering

## Summary

Quality-control and filter scRNA-seq count matrices (e.g., GSE117872 OSCC Cisplatin data) by per-cell gene counts, per-gene cell coverage, mitochondrial percentage, and highly-variable-gene dispersion before feeding them into the scDEAL deep transfer learning pipeline.

## When to use

Use when raw scRNA-seq count data (e.g., GEO datasets like GSE117872) must be cleaned prior to bulk-to-single-cell drug response prediction, and cells/genes failing QC thresholds must be removed without discarding heterogeneity.

## When NOT to use

- Input is already a preprocessed, QC-filtered matrix or a trained-model output such as an AnnData .h5ad with sens_label — re-filtering risks removing signal
- The analysis target is bulk RNA-seq drug response data (GDSC/CCLE cell-line matrices), which scDEAL consumes directly in bulkmodel.py without single-cell QC filters
- Non-human scRNA-seq data requiring homolog-gene mapping first (e.g., the mouse GSE110894 case where only homologous genes were kept before the human-trained model was applied)

## Inputs

- Preprocessed scRNA-seq count matrix for a GEO dataset (e.g., data/GSE117872 directory from scDEAL.zip)
- Bulk drug-screening reference files (ALL_expression.csv, ALL_label_binary_wf.csv)
- Filter parameter settings (min_n_genes, max_n_genes, min_g, min_c, percent_mito, var_genes_disp)

## Outputs

- Filtered scRNA-seq expression matrix retaining cells with >=200 genes and genes expressed in >=3 cells
- AnnData .h5ad object (e.g., save/adata/data/GSE117872[timestamp].h5ad) with adata.obs['sens_label'] for downstream prediction

## How to apply

Run the Scanpy-based filtering utilities shipped with scDEAL (scanpypip) before bulkmodel.py/scmodel.py training. Apply the default cell filters: minimum 200 genes detected per cell (UMI counts >1, --min_g 200) and a maximum of 20,000 genes (--max_n_genes 20000), with --min_n_genes 0 as the lower default bound; drop genes expressed in fewer than 3 cells (--min_c 3); leave --percent_mito at its default 100 unless mitochondrial QC is desired. For highly variable gene selection, set --var_genes_disp (if None, all genes are selected). These thresholds are the scDEAL defaults documented in the methods; adjusted values can be grid-searched per dataset, as the authors did to optimize each of the six benchmark datasets (Data 1-6). The filtered matrix feeds dimensionality reduction (DAE encoder, bottleneck 512 for GSE117872HN120) and downstream AnnData (.h5ad) outputs with adata.obs['sens_label'] (0 = resistance, 1 = sensitivity).

## Related tools

- **Scanpy** (Provides the AnnData-based filtering/preprocessing utilities used to apply min_g/min_c/percent_mito and highly-variable-gene dispersion filters) — https://github.com/OSU-BMBL/scDEAL.git
- **AnnData** (Container format for filtered single-cell data and for the final output .h5ad with sens_label in adata.obs) — https://github.com/OSU-BMBL/scDEAL.git
- **scDEAL** (Framework whose scanpypip utilities expose the filtering parameters (min_n_genes, max_n_genes, min_g, min_c, percent_mito, var_genes_disp) applied before model training) — https://github.com/OSU-BMBL/scDEAL.git

## Examples

```
python scanpypip/preprocessing.py --sc_data GSE117872 --min_g 200 --min_c 3 --max_n_genes 20000 --percent_mito 100
```

## Evaluation signals

- Filtered AnnData .h5ad exists at save/adata/data/ (e.g., GSE117872[timestamp].h5ad) and adata.obs contains a 'sens_label' field encoding 0 = resistance and 1 = sensitivity
- Cells retained satisfy the minimum-gene threshold (>=200 genes with UMI counts >1 by default; --min_g 200) and genes retained are expressed in >=3 cells (--min_c 3)
- No cell exceeds the max_n_genes cap (default 20000) and percent_mito does not exceed the configured threshold (default 100)
- Downstream bulk and single-cell model training completes with encoder/predictor dimensions identical across steps (e.g., bulk_h_dims 256,128; predictor_h_dims 128,64; bottleneck 512 for GSE117872HN120)
- Critical-gene GO enrichment on filtered-data predictions recovers reported terms such as 'DNA repair' (adj. p = 0.039) and 'cell division' (adj. p = 0.003)

## Limitations

- No software version numbers are stated for tools or the environment, so exact filter behavior may vary across Scanpy releases
- Filter defaults (e.g., percent_mito = 100) do not enforce mitochondrial QC; suboptimal thresholds require per-dataset grid-search optimization as the authors performed
- Prediction accuracy could vary depending on the collection of bulk gene expression of cell lines, and more experimentally validated drug response scRNA-seq data would be needed to determine better model hyperparameters

## Evidence

- [methods] min_g: "Minimum number of genes for a cell >1 for filtering propose, default: 200"
- [methods] min_c: "Minimum number of cell that each gene express for filtering propose, default: 3"
- [methods] max_n_genes: "Maximum number of genes for a cell that have UMI counts >1 for filtering propose, default: 20000"
- [methods] var_genes_disp: "Dispersion of highly variable genes selection when pre-processing the data. If None, all genes will be selected"
- [readme] output format: "The expected output format of scDEAL is the"
