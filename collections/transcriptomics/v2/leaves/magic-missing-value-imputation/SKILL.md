---
name: magic-missing-value-imputation
description: Use when an scRNA-seq count matrix fed into the scDrug pipeline is too
  sparse/noisy for reliable clustering and the analyst opts into imputation via the
  --impute flag; default behavior is no imputation.
when_to_use_negative:
- 'When the count matrix is dense enough for the 20-PC neighborhood graph — the scDrug
  default is ''do imputation. default: no'', so imputation should not be applied reflexively'
- When the goal is only drug response prediction from an already-clustered scanpyobj.h5ad
  (drug_response_prediction.py takes the h5ad directly)
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MAGIC
  role: Imputation engine integrated by scDrug to impute missing values in the preprocessed
    scRNA-seq matrix
  repo: https://github.com/ailabstw/scDrug
- name: scDrug
  role: Wrapping pipeline exposing imputation via the --impute flag of single_cell_analysis.py
    inside a Docker container tagged sc-drug
  repo: https://github.com/ailabstw/scDrug
- name: Scanpy
  role: Handles the data preprocessing (filtering, normalization, HVG annotation,
    scaling) that precedes and follows the imputed matrix
  repo: https://github.com/ailabstw/scDrug
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:7e743e8fee53e5b17c76fe4cbac271ddccb030683253d0d8efe08cc298fcce2d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - MAGIC
  - Python
  - Scanpy
  - scDrug
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# magic-missing-value-imputation

## Summary

Optional MAGIC-based imputation step inside the scDrug Single-Cell Data Analysis module, which fills missing values in scRNA-seq count matrices (10x mtx or CSV input) after Scanpy preprocessing so downstream PCA, Harmony correction, and Louvain clustering operate on a denser expression object.

## When to use

Use when an scRNA-seq count matrix fed into the scDrug pipeline is too sparse/noisy for reliable clustering and the analyst opts into imputation via the --impute flag; default behavior is no imputation.

## When NOT to use

- When the count matrix is dense enough for the 20-PC neighborhood graph — the scDrug default is 'do imputation. default: no', so imputation should not be applied reflexively
- When the goal is only drug response prediction from an already-clustered scanpyobj.h5ad (drug_response_prediction.py takes the h5ad directly)

## Inputs

- 10x-Genomics-formatted mtx directory (matrix.mtx, genes.tsv, barcodes.tsv) or CSV count matrix
- Optional Scanpy AnnData object (h5ad) for sub-clustering with --clusters
- Optional metadata CSV for batch correction (index in first column)

## Outputs

- Imputed Scanpy AnnData object scanpyobj.h5ad with louvain assignments
- UMAP plot umap_cluster.png
- Differentially expressed genes cluster_DEGs.csv
- Gene expression profile file GEP.txt (with --GEP)

## How to apply

Run scDrug's single_cell_analysis.py on a 10x-Genomics-formatted mtx directory or CSV with the --impute flag, which triggers MAGIC integration after the Scanpy preprocessing chain (filter cells with <200 genes, genes in <3 cells, mitochondrial proportion below 30%; normalize to 10,000 counts, log transform, scale). Imputation is explicitly optional — the pipeline's default is 'no' — so decide whether the downstream graph build from the top 20 PCs needs a denser matrix. The imputed object feeds PCA, optional Harmony batch correction using the batch column (e.g. 'PatientID'), and Louvain clustering (resolution default 0.6, or --auto-resolution over [0.4, 1.4]). The final imputed and clustered state is serialized as scanpyobj.h5ad alongside umap_cluster.png and cluster_DEGs.csv.

## Related tools

- **MAGIC** (Imputation engine integrated by scDrug to impute missing values in the preprocessed scRNA-seq matrix) — https://github.com/ailabstw/scDrug
- **scDrug** (Wrapping pipeline exposing imputation via the --impute flag of single_cell_analysis.py inside a Docker container tagged sc-drug) — https://github.com/ailabstw/scDrug
- **Scanpy** (Handles the data preprocessing (filtering, normalization, HVG annotation, scaling) that precedes and follows the imputed matrix) — https://github.com/ailabstw/scDrug

## Examples

```
python3 single_cell_analysis.py -i INPUT --impute -o ./output
```

## Evaluation signals

- scanpyobj.h5ad exists and loads as a valid AnnData object with a categorical 'louvain' observation field
- Chosen clustering resolution (default 0.6, or auto-selected) lies within the [0.4, 1.4] interval scanned with 0.2 spacing
- Imputed values appear in the expression matrix only when --impute was passed; the default run leaves imputation off
- Downstream UMAP (umap_cluster.png) renders clusters consistent with the Louvain assignments stored in the h5ad
- Input QC thresholds were enforced: cells <200 genes and genes in <3 cells dropped, mitochondrial proportion kept below 30%

## Limitations

- Imputation is optional and off by default; the article states it is applied 'once the data imputation is needed', leaving the decision to the analyst
- scDrug is restricted to a Python environment and Docker-in-Docker execution; if docker-in-docker is unavailable the CIBERSORTx container must be run outside with a shared mount
- Prediction of drug response in scDrug is limited to experimentally screened compounds, and available scRNA-seq data with drug treatments are too short for evaluation

## Evidence

- [methods] MAGIC integrated as imputation step: "Once the data imputation is needed, scDrug also integrates MAGIC [37] to impute missing values."
- [methods] Imputation sits between Scanpy preprocessing and Harmony batch correction: "including data preprocessing by Scanpy [19], imputation by MAGIC [37], batch correction by Harmony [20]"
- [readme] Imputation is optional and off by default in the CLI: "--impute              do imputation. default: no"
- [methods] Preprocessing thresholds applied before imputation: "we filtered out cells with less than 200 genes expressed and genes expressed in less than three cells and kept the cells with a proportion of mitochondrial genes below 30 %"
- [readme] Imputed object feeds clustering and downstream outputs: "performed fundamental data analysis, and output a Scanpy Anndata object `scanpyobj.h5ad`, a UMAP `umap_cluster.png`"
