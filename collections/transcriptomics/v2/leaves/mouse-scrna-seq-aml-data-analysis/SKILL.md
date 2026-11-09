---
name: mouse-scrna-seq-aml-data-analysis
description: Use when analyzing drug-treated mouse scRNA-seq (e.g., Mus musculus AML
  cells, GSE110894, I-BET-762) with a human-trained scDEAL model, or when correlating
  predicted response probabilities with Monocle3 pseudotime.
when_to_use_negative:
- Data is human scRNA-seq (Datasets 1-5, e.g., GSE117872/GSE112274) rather than mouse
  AML — no homologous-gene restriction is needed and other case-study workflows apply
- Goal is bulk-level drug response prediction from cell-line data (GDSC/CCLE) rather
  than single-cell trajectory analysis of a treated mouse dataset
- No pretrained AnnData or scDEAL model checkpoint for GSE110894 is available and
  the user is not prepared to run bulkmodel.py/scmodel.py training first
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDEAL
  role: Predicts drug response probabilities on the mouse AML cells and supplies trajectory.py
    for Monocle3-style pseudotime analysis
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: Scanpy
  role: Loads and manipulates the AnnData (.h5ad) object containing the GSE110894
    predictions
- name: AnnData
  role: File format (.h5ad) holding the predicted response labels and pseudotime for
    the Data 6 cells
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:ad2d8d27feb38c01e04a20652d02ed3f67cdf9523cf3304fbd969b121f6daa30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - Scanpy
  - scDEAL
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# mouse-scrna-seq-aml-data-analysis

## Summary

Case-study skill for analyzing the mouse AML scRNA-seq dataset (Data 6, GSE110894, I-BET-762, 1419 cells) with scDEAL, including trans-species prediction on homologous genes and pseudotime correlation of predicted response probabilities.

## When to use

Use when analyzing drug-treated mouse scRNA-seq (e.g., Mus musculus AML cells, GSE110894, I-BET-762) with a human-trained scDEAL model, or when correlating predicted response probabilities with Monocle3 pseudotime.

## When NOT to use

- Data is human scRNA-seq (Datasets 1-5, e.g., GSE117872/GSE112274) rather than mouse AML — no homologous-gene restriction is needed and other case-study workflows apply
- Goal is bulk-level drug response prediction from cell-line data (GDSC/CCLE) rather than single-cell trajectory analysis of a treated mouse dataset
- No pretrained AnnData or scDEAL model checkpoint for GSE110894 is available and the user is not prepared to run bulkmodel.py/scmodel.py training first

## Inputs

- adata.zip containing pre-trained AnnData (.h5ad) object for GSE110894 (Data 6)
- Mouse scRNA-seq expression data from GSE110894 (Bell et al., I-BET-762, 1419 Mus musculus AML cells)
- scDEAL-predicted response probabilities in adata.obs['sens_label']
- scDEAL trajectory.py utilities and casestudy/analysis_tarined_anndata.ipynb template

## Outputs

- pseudotime_correlations.csv with resistant and sensitive Pearson correlation values
- Regression plot of pseudotime versus response score (Fig. 5e-f style)
- Monocle3-style pseudotime trajectory inference results on the Data 6 cells

## How to apply

Load the pre-trained AnnData result for GSE110894 (1419 mouse AML cells treated with I-BET.762) from adata.zip via scanpy.read_h5ad, and extract scDEAL-predicted response probabilities from adata.obs['sens_label'] (0 = resistant, 1 = sensitive). Because only homologous genes were kept in the mouse data while the scDEAL model was trained with human cancer cell lines, restrict the expression matrix to orthologs before prediction. Run Monocle3-style pseudotime trajectory inference using scDEAL's trajectory.py utilities on Scanpy/AnnData objects, then compute Pearson correlations between pseudotime and the predicted resistant/sensitive probability scores, matching Fig. 5e-f and Source Data 9-10. Generate the regression plot of pseudotime versus response score as in the casestudy/analysis_tarined_anndata.ipynb template and write correlation values to pseudotime_correlations.csv.

## Related tools

- **scDEAL** (Predicts drug response probabilities on the mouse AML cells and supplies trajectory.py for Monocle3-style pseudotime analysis) — https://github.com/OSU-BMBL/scDEAL.git
- **Scanpy** (Loads and manipulates the AnnData (.h5ad) object containing the GSE110894 predictions)
- **AnnData** (File format (.h5ad) holding the predicted response labels and pseudotime for the Data 6 cells)

## Examples

```
python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --lr 0.5 --sampling "upsampling"
```

## Evaluation signals

- Pearson correlation between pseudotime and resistant probability matches the reported +0.81 within ±0.05
- Pearson correlation between pseudotime and sensitive probability matches the reported -0.93 within ±0.05
- pseudotime_correlations.csv exists and contains the required correlation fields
- adata.obs['sens_label'] contains binary labels (0 = resistant, 1 = sensitive) for all 1419 cells
- Regression plot reproduces the trend shown in Fig. 5e-f / Source Data 9-10

## Limitations

- Only homologous genes were kept in the mouse data while the scDEAL model was trained with human cancer cell lines
- Trans-species (human-to-mouse) reliability could not be systematically evaluated due to the limited number of drug-treated mouse benchmark scRNA-seq data in the public domain
- Prediction accuracy could vary depending on the collection of bulk gene expression of cell lines

## Evidence

- [discussion] In our case study of mouse scRNA-seq data (Data 6), only homologous genes were kept in the mouse data and the scDEAL model was still trained with human cancer cell lines.: "In our case study of mouse scRNA-seq data (Data 6), only homologous genes were kept in the mouse data and the scDEAL model was still trained with human cancer cell lines."
- [methods] The expected output format of scDEAL is the AnnData object (.h5ad) applied by the scanpy package: [evidence span withheld — anchor status: not_found]
- [readme] This step is a built-in testing case of acute myeloid leukemia cells [Bell et al.] accessed from Gene Expression Omnibus (GEO) accession GSE110894.: [evidence span withheld — anchor status: not_found]
- [discussion] Due to the limited number of drug-treated mouse benchmark scRNA-seq data in the public domain, we could not systematically evaluate and optimize the trans-species reliability in the current study.: "Due to the limited number of drug-treated mouse benchmark scRNA-seq data in the public domain, we could not systematically evaluate and optimize the trans-species reliability"
