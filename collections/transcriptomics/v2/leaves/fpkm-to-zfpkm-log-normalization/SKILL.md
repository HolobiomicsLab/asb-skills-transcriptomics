---
name: fpkm-to-zfpkm-log-normalization
description: Use when normalized RNA-Seq FPKM expression matrices (e.g., TCGA KIRC/LIHC/PRAD
  protein-coding and long non-coding genes) must be reduced to actively expressed
  genes before building co-expression networks.
when_to_use_negative:
- Input is raw RNA-Seq counts rather than normalized FPKM values — differential expression
  in this pipeline uses the DESeq2 R/Bioconductor package, not zFPKM.
- Data are miRNA expression rather than protein-coding/long non-coding RNA genes —
  miRNAs are pre-processed separately in this pipeline (on average 81% of miRNAs are
  extracted) and the zFPKM > −3.0 filter as described applies to RNA genes.
- The goal is calling differentially expressed genes for a gene signature — use DESeq2
  instead; zFPKM filtering only removes background/noisy genes as input selection
  for network construction.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Implements the zFPKM transformation (z-score of log-FPKM per gene/sample)
    and the mean zFPKM > −3.0 active-gene filter within the preprocessing scripts
    of Pipeline.ipynb
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/preprocessing.py
- name: GMQL
  role: Retrieves the TCGA gene expression data (GRCh38) that the zFPKM filtering
    is applied to
  repo: http://gmql.eu
- name: PyGMQL
  role: Python interface through which the TCGA gene expression data fed into the
    zFPKM filter are retrieved
  repo: https://pygmql.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:2d089f2e1aacd0ef547e4b1d82ae619fc8363ceea5df2928af19f5bf60b77d7c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GMQL
  - Jupyter Notebook
  - PyGMQL
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# fpkm-to-zfpkm-log-normalization

## Summary

Transforms normalized RNA-Seq FPKM gene expression values into zFPKM (z-score of log-FPKM, Hart et al. approach) and filters active genes by keeping only those whose mean zFPKM exceeds −3.0, discarding background/noisy transcripts before gene co-expression network construction.

## When to use

Use when normalized RNA-Seq FPKM expression matrices (e.g., TCGA KIRC/LIHC/PRAD protein-coding and long non-coding genes) must be reduced to actively expressed genes before building co-expression networks.

## When NOT to use

- Input is raw RNA-Seq counts rather than normalized FPKM values — differential expression in this pipeline uses the DESeq2 R/Bioconductor package, not zFPKM.
- Data are miRNA expression rather than protein-coding/long non-coding RNA genes — miRNAs are pre-processed separately in this pipeline (on average 81% of miRNAs are extracted) and the zFPKM > −3.0 filter as described applies to RNA genes.
- The goal is calling differentially expressed genes for a gene signature — use DESeq2 instead; zFPKM filtering only removes background/noisy genes as input selection for network construction.

## Inputs

- TCGA RNA-Seq FPKM gene expression matrices for normal and cancer samples (KIRC, LIHC, PRAD), GRCh38 assembly, protein-coding and long non-coding genes only, retrieved via GMQL/PyGMQL
- Gene/sample FPKM value table in tabular format (e.g., CSV)

## Outputs

- Table of retained active genes with their zFPKM values (e.g., active_genes_zfpkm.csv with a zFPKM field per gene)
- Filtered gene expression matrix restricted to genes with mean zFPKM > −3.0 in both normal and cancer conditions, used for building co-expression networks

## How to apply

Load the TCGA RNA-Seq FPKM gene expression matrices for the normal and cancer samples (retrieved via GMQL/PyGMQL for the GRCh38 assembly, restricted to protein-coding and long non-coding genes). For each gene/sample pair, compute zFPKM as the z-score of the log(FPKM) values following the Hart et al. normalization approach. Compute the mean zFPKM per gene within each condition (normal and cancer) and retain genes whose mean zFPKM is higher than −3.0 in BOTH conditions, discarding background/noisy genes. This preprocessing step (part of preprocessing.py / Pipeline.ipynb) outputs the retained genes with their zFPKM values (e.g., active_genes_zfpkm.csv), which become the input space for the subsequent co-expression network construction. On average ~20% of RNA (protein-coding or long non-coding) genes pass this filter.

## Related tools

- **Python** (Implements the zFPKM transformation (z-score of log-FPKM per gene/sample) and the mean zFPKM > −3.0 active-gene filter within the preprocessing scripts of Pipeline.ipynb) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/preprocessing.py
- **GMQL** (Retrieves the TCGA gene expression data (GRCh38) that the zFPKM filtering is applied to) — http://gmql.eu
- **PyGMQL** (Python interface through which the TCGA gene expression data fed into the zFPKM filter are retrieved) — https://pygmql.readthedocs.io/en/latest/

## Examples

```
mean_z = genes.groupby('gene')['zFPKM'].mean(); kept = mean_z[(mean_z.normal > -3.0) & (mean_z.cancer > -3.0)]; kept.to_csv('active_genes_zfpkm.csv')
```

## Evaluation signals

- Every retained gene has mean zFPKM > −3.0 in both the normal and the cancer condition.
- The output file (e.g., active_genes_zfpkm.csv) exists and contains a zFPKM field for each gene.
- Per-gene zFPKM values are z-scores of log(FPKM) across samples, so their cross-sample distribution is centered near 0 with standard deviation 1.
- The proportion of retained genes is in the expected range: on average ~20% of RNA (protein-coding or long non-coding) genes pass this filter.
- The downstream pipeline executes without errors on the filtered matrix, i.e., the retained genes form the input of the co-expression network construction step.

## Limitations

- The −3.0 threshold was validated on TCGA FPKM data for KIRC, LIHC, and PRAD; no calibration procedure for other datasets or expression scales is described.
- Lowly expressed but potentially biologically relevant genes are permanently discarded before network construction, since only genes above the zFPKM threshold proceed to the co-expression networks.
- The filter targets RNA genes (protein-coding and long non-coding); miRNAs follow a separate preprocessing path in the pipeline.

## Evidence

- [methods] The selected RNA-Seq data have a mean value of normalized fragments per kilobase of transcript per million mapped reads (zFPKM) higher than −3.0: "The selected RNA-Seq data have a mean value of normalized fragments per kilobase of transcript per million mapped reads (zFPKM) higher than −3.0"
- [results] On average, 20% of RNA (protein-coding or long non-coding) genes and 81% of miRNAs are extracted: "On average, 20% of RNA (protein-coding or long non-coding) genes and 81% of miRNAs are extracted"
- [readme] that contain the TCGA gene expression data retrieved through [GMQL](http://gmql.eu) and [PyGMQL](https://pygmql.readthedocs.io/en/latest/): "that contain the TCGA gene expression data retrieved through [GMQL](http://gmql.eu) and [PyGMQL](https://pygmql.readthedocs.io/en/latest/)"
