---
name: tcga-rnaseq-matrix-construction
description: Use when TCGA RNA-Seq FPKM data must be turned into normalized gene expression
  matrices for network analysis, and background/noisy genes must be removed via a
  mean zFPKM > -3.0 activity threshold.
when_to_use_negative:
- The input data are miRNA expression profiles, which follow a separate pre-processing
  track in this pipeline (81% of miRNAs extracted on average, not the zFPKM gene filter)
- The goal is differential expression analysis with DESeq2, which is used in this
  article as a comparison method rather than the active-gene matrix construction route
- Expression data are not from TCGA / not FPKM-based counts retrievable via GMQL,
  so the zFPKM thresholding rationale does not directly apply
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GMQL
  role: Query language used to retrieve the TCGA gene expression data and metadata
    from the TCGA repository
  repo: http://gmql.eu
- name: PyGMQL
  role: Python interface through which the TCGA gene expression data are loaded into
    the pipeline
  repo: https://pygmql.readthedocs.io/en/latest/
- name: Python
  role: Implementation language for preprocessing.py and creation_matrices.py, which
    compute zFPKM and build the expression matrices
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Pipeline.ipynb contains the complete pipeline including the matrix construction
    and active-gene extraction steps
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:1d51bdbbccef5243a43f31c12c1a5d1987338bbc5bda44b5cc9b449a15f46c76
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GMQL
  - Jupyter Notebook
  - PyGMQL
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# tcga-rnaseq-matrix-construction

## Summary

Retrieves TCGA RNA-Seq gene expression data (protein-coding and long non-coding genes) via GMQL/PyGMQL for normal and cancer conditions and builds zFPKM-normalized expression matrices, retaining only actively expressed genes as input for gene co-expression network construction.

## When to use

Use when TCGA RNA-Seq FPKM data must be turned into normalized gene expression matrices for network analysis, and background/noisy genes must be removed via a mean zFPKM > -3.0 activity threshold.

## When NOT to use

- The input data are miRNA expression profiles, which follow a separate pre-processing track in this pipeline (81% of miRNAs extracted on average, not the zFPKM gene filter)
- The goal is differential expression analysis with DESeq2, which is used in this article as a comparison method rather than the active-gene matrix construction route
- Expression data are not from TCGA / not FPKM-based counts retrievable via GMQL, so the zFPKM thresholding rationale does not directly apply

## Inputs

- TCGA RNA-Seq FPKM gene expression data for KIRC, LIHC, and PRAD (normal and cancer samples), retrieved via GMQL for the GRCh38 assembly
- Gene annotation restricting the analysis to protein-coding and long non-coding RNA genes

## Outputs

- Normalized gene expression matrices (gene x sample) written to the Datasets_created folder
- active_genes_zfpkm.csv containing the retained active genes and their zFPKM values

## How to apply

Retrieve the TCGA RNA-Seq FPKM gene expression matrices for the normal and cancer samples of KIRC, LIHC, and PRAD through GMQL/PyGMQL for the GRCh38 assembly, restricted to protein-coding and long non-coding RNA genes. Compute zFPKM for each gene/sample pair as the z-score of log(FPKM) values following the Hart et al. normalization approach. Compute the mean zFPKM per gene within each condition (normal and cancer), then keep only genes whose mean zFPKM is higher than -3.0 in both conditions, discarding background/noisy genes — the paper reports on average ~20% of RNA genes pass this pre-processing extraction. Write the retained genes and their zFPKM values to active_genes_zfpkm.csv and the resulting matrices to the Datasets_created folder, as implemented in preprocessing.py/creation_matrices.py and the Pipeline.ipynb notebook.

## Related tools

- **GMQL** (Query language used to retrieve the TCGA gene expression data and metadata from the TCGA repository) — http://gmql.eu
- **PyGMQL** (Python interface through which the TCGA gene expression data are loaded into the pipeline) — https://pygmql.readthedocs.io/en/latest/
- **Python** (Implementation language for preprocessing.py and creation_matrices.py, which compute zFPKM and build the expression matrices) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Pipeline.ipynb contains the complete pipeline including the matrix construction and active-gene extraction steps) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb

## Examples

```
pip install -r requirements.txt && jupyter notebook Pipeline.ipynb
```

## Evaluation signals

- The script runs successfully and active_genes_zfpkm.csv exists with a zFPKM field per gene
- Every retained gene has mean zFPKM > -3.0 in BOTH the normal and the cancer condition
- On average ~20% of RNA (protein-coding or long non-coding) genes are retained, matching the paper's reported pre-processing extraction rate
- Resulting matrices contain only protein-coding and long non-coding genes, retrieved from the GRCh38 assembly via GMQL

## Limitations

- The zFPKM activity filter is aggressive: on average about 80% of RNA genes are discarded as background/noisy
- The public repository ships data, scripts and results only for the LIHC tumor analysis, so KIRC and PRAD matrices must be regenerated by the user
- The -3.0 cutoff follows the Hart et al. zFPKM approach on normalized FPKM values; it may not transfer to non-FPKM or non-TCGA expression data

## Evidence

- [methods] TCGA retrieval and matrix construction: "The selected RNA-Seq data have a mean value of normalized fragments per kilobase of transcript per million mapped reads (zFPKM) higher than −3.0"
- [methods] Data source via GMQL: "from the TCGA repository17 using the GenoMetric Query Language (GMQL), an innovative query language for genomic data and metadata25"
- [results] Expected extraction rate: "On average, 20% of RNA (protein-coding or long non-coding) genes and 81% of miRNAs are extracted"
- [readme] Repository tooling for data retrieval: "the TCGA gene expression data retrieved through [GMQL](http://gmql.eu) and [PyGMQL](https://pygmql.readthedocs.io/en/latest/)"
