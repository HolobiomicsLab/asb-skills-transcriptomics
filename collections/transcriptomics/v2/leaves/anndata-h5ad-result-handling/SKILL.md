---
name: anndata-h5ad-result-handling
description: Use when a scDEAL run (bulkmodel.py/scmodel.py) must produce, load, or
  validate its single-cell output as a scanpy-readable .h5ad AnnData file under save/adata,
  or when downstream notebooks need cell-level sens_label predictions.
when_to_use_negative:
- When only bulk-level model artifacts (save/bulk_pre checkpoints) are needed and
  no single-cell AnnData output is produced
- When the goal is raw preprocessing/filtering of scRNA-seq (min_g, min_c, percent_mito)
  rather than handling the scored result object
tools:
- name: scDEAL
  role: Produces the single-cell drug response predictions that are serialized into
    the AnnData .h5ad output
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: Scanpy
  role: Package used to apply and read the AnnData (.h5ad) result object in downstream
    analysis
  repo: https://anndata.readthedocs.io/en/latest/anndata.AnnData.html
- name: AnnData
  role: Defines the required .h5ad object format for scDEAL outputs, including adata.obs
    fields like sens_label
  repo: https://anndata.readthedocs.io/en/latest/anndata.AnnData.html
provenance:
  source_task_ids:
  - task_006
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:dfb16e95da29f8e73f55788baa326f687a169d0557183ec071bf26d212f53e34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - Scanpy
  - conda-pack
  - scDEAL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# anndata-h5ad-result-handling

## Summary

Consumes and writes scDEAL single-cell drug response prediction results as AnnData (.h5ad) objects via scanpy, preserving cell-level predictions such as adata.obs['sens_label'] (0 = resistance, 1 = sensitivity) and integrated-gradient critical gene results for downstream validation and case-study analysis.

## When to use

Use when a scDEAL run (bulkmodel.py/scmodel.py) must produce, load, or validate its single-cell output as a scanpy-readable .h5ad AnnData file under save/adata, or when downstream notebooks need cell-level sens_label predictions.

## When NOT to use

- When only bulk-level model artifacts (save/bulk_pre checkpoints) are needed and no single-cell AnnData output is produced
- When the goal is raw preprocessing/filtering of scRNA-seq (min_g, min_c, percent_mito) rather than handling the scored result object

## Inputs

- Single-cell model output from scmodel.py (predictions on scRNA-seq data, e.g. GSE110894 or GSE117872)
- adata.obs['sens_label'] drug sensitivity labels (0 = resistance, 1 = sensitivity)
- Integrated gradient (IG) scores for critical gene identification passed into analysis_criticalgenes.ipynb

## Outputs

- AnnData object saved as .h5ad under save/adata/data/ (e.g. GSE117872[timestamp].h5ad) with adata.obs['sens_label'] in {0,1}
- scanpy-loaded AnnData usable for GO enrichment of critical genes in the analysis_criticalgenes.ipynb case study

## How to apply

After running scmodel.py, the single-cell predictions are serialized as an AnnData object (.h5ad) with the single-cell drug sensitivity stored in adata.obs['sens_label'], where 0 encodes resistance and 1 encodes sensitivity, and saved under save/adata/data (e.g. GSE117872[timestamp].h5ad). Load the object with scanpy (sc.read_h5ad) in the scDEALenv conda-pack environment for downstream steps such as the analysis_criticalgenes.ipynb case study on integrated-gradient critical genes. The encoder/predictor dimensions must be identical between bulkmodel.py and scmodel.py (e.g. --bulk_h_dims "256,128", --predictor_h_dims "128,64", bottleneck 512) so the checkpoint loads cleanly and the resulting h5ad is valid. Validation is schema-level: the file must exist, parse with scanpy, and expose sens_label in adata.obs with values restricted to {0,1}.

## Related tools

- **scDEAL** (Produces the single-cell drug response predictions that are serialized into the AnnData .h5ad output) — https://github.com/OSU-BMBL/scDEAL.git
- **Scanpy** (Package used to apply and read the AnnData (.h5ad) result object in downstream analysis) — https://anndata.readthedocs.io/en/latest/anndata.AnnData.html
- **AnnData** (Defines the required .h5ad object format for scDEAL outputs, including adata.obs fields like sens_label) — https://anndata.readthedocs.io/en/latest/anndata.AnnData.html

## Examples

```
source scDEALenv/bin/activate; python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --lr 0.5 --sampling "upsampling" --printgene "F" -mod "new"; import scanpy as sc; adata = sc.read_h5ad("save/adata/data/GSE117872.h5ad"); adata.obs["sens_label"]
```

## Evaluation signals

- The .h5ad file exists under save/adata/data/ (e.g. GSE117872[timestamp].h5ad) after scmodel.py completes
- The file loads with scanpy without errors (sc.read_h5ad returns a valid AnnData object)
- adata.obs contains a 'sens_label' column whose values are all in {0,1} (0 = resistance, 1 = sensitivity)
- The saved AnnData carries the integrated gradient scores needed by analysis_criticalgenes.ipynb to reproduce reported GO terms ('DNA repair' adj. p = 0.039; 'cell division' adj. p = 0.003)
- Checkpoint-based runs use identical encoder/predictor dimensions in both steps (--bulk_h_dims/--encoder_h_dims '256,128', --predictor_h_dims '128,64', same bottleneck), otherwise the model fails to load and no h5ad is produced

## Limitations

- scDEAL prediction accuracy depends on the collection of bulk gene expression of cell lines used in training
- No software version numbers are stated for tools or the environment, so h5ad compatibility across scanpy/anndata versions is not guaranteed
- The README output section is truncated, so full details of additional AnnData slots (beyond sens_label) are not documented

## Evidence

- [methods] The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad): "The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad)"
- [readme] The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad) applied by the scanpy package: "The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad) applied by the scanpy package"
- [readme] Add trained adata.h5ad objects for 6 data examples.: "Add trained adata.h5ad objects for 6 data examples."
- [readme] Remember that the dimension of the encoder and predictor should be identical (--encoder_h_dims(bulk_h_dims) "256,128", --predictor_h_dims "128,64", --bottleneck 256) in two steps.: "Remember that the dimension of the encoder and predictor should be identical (--encoder_h_dims(bulk_h_dims) "256,128", --predictor_h_dims "128,64", --bottleneck 256) in two steps."
