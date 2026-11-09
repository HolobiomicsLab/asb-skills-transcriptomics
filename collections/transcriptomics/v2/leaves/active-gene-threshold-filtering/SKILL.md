---
name: active-gene-threshold-filtering
description: Use when normalized RNA-Seq FPKM matrices (e.g. TCGA KIRC/LIHC/PRAD,
  protein-coding and lncRNA genes) must be reduced to actively expressed genes prior
  to building gene co-expression networks.
when_to_use_negative:
- Data are miRNA expression profiles — the paper reports a much higher pass rate for
  miRNAs (81% vs ~20% of RNA genes), so the −3.0 zFPKM cutoff is not calibrated for
  them
- Input is raw unnormalized counts with no FPKM normalization — zFPKM requires fragments-per-kilobase-million
  values whose log can be z-scored
- The goal is differential expression analysis (handled separately in this pipeline
  with DESeq2), not selection of a common actively expressed background gene set
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Python
  role: Implements the zFPKM transformation, per-condition mean computation, and −3.0
    thresholding in the preprocessing step (preprocessing.py)
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/preprocessing.py
- name: GMQL
  role: Retrieves the TCGA gene expression data (FPKM matrices) that the zFPKM filter
    is applied to
  repo: http://gmql.eu
- name: PyGMQL
  role: Python interface used to fetch the TCGA datasets consumed by the preprocessing/active-gene
    filtering step
  repo: https://pygmql.readthedocs.io/en/latest/
- name: Jupyter Notebook
  role: Pipeline.ipynb executes the preprocessing stage containing the zFPKM active-gene
    selection
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:ae5fd2e3901da7bb3094a665113ae7cb2e756f19ace920c0540afb9183ee694f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GMQL
  - Jupyter Notebook
  - PyGMQL
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# active-gene-threshold-filtering

## Summary

Selects actively expressed RNA-Seq genes by computing zFPKM (z-score of log FPKM, Hart et al. approach) for each gene/sample pair and retaining only genes whose mean zFPKM exceeds −3.0 in both the normal and cancer conditions, discarding background/noisy genes before co-expression network construction.

## When to use

Use when normalized RNA-Seq FPKM matrices (e.g. TCGA KIRC/LIHC/PRAD, protein-coding and lncRNA genes) must be reduced to actively expressed genes prior to building gene co-expression networks.

## When NOT to use

- Data are miRNA expression profiles — the paper reports a much higher pass rate for miRNAs (81% vs ~20% of RNA genes), so the −3.0 zFPKM cutoff is not calibrated for them
- Input is raw unnormalized counts with no FPKM normalization — zFPKM requires fragments-per-kilobase-million values whose log can be z-scored
- The goal is differential expression analysis (handled separately in this pipeline with DESeq2), not selection of a common actively expressed background gene set

## Inputs

- TCGA RNA-Seq FPKM gene expression matrices for normal and cancer samples (KIRC, LIHC, PRAD), GRCh38 assembly, protein-coding and long non-coding genes only
- Gene/sample expression table retrieved through GMQL/PyGMQL

## Outputs

- Table of retained active genes with their zFPKM values (active_genes_zfpkm.csv)
- Filtered gene set used as input for co-expression network construction

## How to apply

Load the TCGA RNA-Seq FPKM gene expression matrices for normal and cancer samples, retrieved via GMQL/PyGMQL for the GRCh38 assembly and restricted to protein-coding and long non-coding genes. Compute zFPKM for each gene/sample pair as the z-score of log(FPKM) values following the Hart et al. normalization approach. Compute the mean zFPKM per gene separately within each condition (normal and cancer). Retain only genes whose mean zFPKM is higher than −3.0 in BOTH conditions, which removes background/noisy transcripts before downstream co-expression network inference. Write the retained genes and their zFPKM values to an output table (active_genes_zfpkm.csv). This is the first thresholding step of the pipeline, implemented in the preprocessing stage (preprocessing.py / Pipeline.ipynb), and is distinct from the later edge-level filters (99th-percentile Euclidean threshold, permutation-based Pearson threshold, disparity filter).

## Related tools

- **Python** (Implements the zFPKM transformation, per-condition mean computation, and −3.0 thresholding in the preprocessing step (preprocessing.py)) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/preprocessing.py
- **GMQL** (Retrieves the TCGA gene expression data (FPKM matrices) that the zFPKM filter is applied to) — http://gmql.eu
- **PyGMQL** (Python interface used to fetch the TCGA datasets consumed by the preprocessing/active-gene filtering step) — https://pygmql.readthedocs.io/en/latest/
- **Jupyter Notebook** (Pipeline.ipynb executes the preprocessing stage containing the zFPKM active-gene selection) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb

## Examples

```
python preprocessing.py  # compute zFPKM (z-score of log FPKM) per gene/sample, keep genes with mean zFPKM > -3.0 in both conditions -> active_genes_zfpkm.csv
```

## Evaluation signals

- Every retained gene has mean zFPKM > −3.0 in BOTH the normal and the cancer condition; any gene violating this in either condition indicates a filtering bug
- Output file active_genes_zfpkm.csv exists and contains a zFPKM field per gene
- Retention rate is on the order of ~20% of RNA (protein-coding or long non-coding) genes, matching the paper's reported average — a much higher rate suggests the threshold was not applied correctly
- Gene set is restricted to protein-coding and long non-coding genes (no miRNAs) from the GRCh38 TCGA retrieval

## Limitations

- The −3.0 cutoff is calibrated for protein-coding/long non-coding RNA genes; miRNAs behave differently (81% retained), so the threshold does not transfer directly
- The filter is applied per condition and a gene must pass in both normal and cancer, so genes active in only one condition are dropped even if biologically relevant to the disease state
- zFPKM requires log(FPKM) values, so zero/non-expressible FPKM entries must be handled before z-scoring; the paper attributes the normalization to the Hart et al. approach without further implementation detail

## Evidence

- [methods] The selected RNA-Seq data have a mean value of normalized fragments per kilobase of transcript per million mapped reads (zFPKM) higher than −3.0: "The selected RNA-Seq data have a mean value of normalized fragments per kilobase of transcript per million mapped reads (zFPKM) higher than −3.0"
- [results] On average, 20% of RNA (protein-coding or long non-coding) genes and 81% of miRNAs are extracted: "On average, 20% of RNA (protein-coding or long non-coding) genes and 81% of miRNAs are extracted"
- [readme] python files which contain the main steps of the process: "preprocessing.py, [creation_matrices.py](https://github.com/DEIB-GECO/GeneNetFusion/blob/master/creation_matrices.py), [fusion_networks.py](https://github.com/DEIB-GECO/GeneNetFusion/blob/master/fusio"
- [readme] TCGA gene expression data retrieved through GMQL and PyGMQL: "that contain the TCGA gene expression data retrieved through [GMQL](http://gmql.eu) and [PyGMQL](https://pygmql.readthedocs.io/en/latest/)"
