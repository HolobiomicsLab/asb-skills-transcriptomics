---
name: gene-expression-data-preprocessing
description: Use when raw TCGA-style FPKM gene expression matrices must be reduced
  to actively expressed genes before co-expression network construction, to remove
  background/noisy genes.
when_to_use_negative:
- The expression data are already filtered to a curated active-gene or biomarker signature
  list
- The analysis requires differential expression (DE) analysis with DESeq2 rather than
  expression-level filtering for network construction
- Input data are miRNA expression rather than protein-coding/long non-coding RNA gene
  expression (a separate 81% miRNA pre-extraction is reported)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GMQL
  role: Query language used to retrieve the TCGA gene expression data and metadata
    from the TCGA repository
  repo: http://gmql.eu
- name: PyGMQL
  role: Python API used together with GMQL to fetch the TCGA gene expression datasets
  repo: https://pygmql.readthedocs.io/en/latest/
- name: Python
  role: Implements the preprocessing steps, including zFPKM computation and the mean
    zFPKM > −3.0 filtering (preprocessing.py)
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/preprocessing.py
- name: Jupyter Notebook
  role: Pipeline.ipynb contains the complete pipeline, including this preprocessing/filtering
    step
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:882346480f3a57239ff63b08bed3a1bf6f93e4b3094c6935246c89d144060bd2
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# gene-expression-data-preprocessing

## Summary

Preprocessing step of the GeneNetFusion pipeline that filters TCGA RNA-Seq gene expression matrices to actively expressed genes by computing zFPKM (Hart et al.) and retaining genes whose mean zFPKM is higher than −3.0 in both normal and cancer conditions.

## When to use

Use when raw TCGA-style FPKM gene expression matrices must be reduced to actively expressed genes before co-expression network construction, to remove background/noisy genes.

## When NOT to use

- The expression data are already filtered to a curated active-gene or biomarker signature list
- The analysis requires differential expression (DE) analysis with DESeq2 rather than expression-level filtering for network construction
- Input data are miRNA expression rather than protein-coding/long non-coding RNA gene expression (a separate 81% miRNA pre-extraction is reported)

## Inputs

- TCGA RNA-Seq FPKM gene expression matrices for normal and cancer samples of KIRC, LIHC, and PRAD, retrieved via GMQL/PyGMQL (GRCh38 assembly)
- Gene annotation restricting analysis to protein-coding and long non-coding RNA genes

## Outputs

- active_genes_zfpkm.csv containing retained genes and their zFPKM values
- Filtered set of actively expressed RNA genes used as input for building the gene co-expression networks

## How to apply

Retrieve TCGA RNA-Seq FPKM gene expression matrices for the normal and cancer samples (KIRC, LIHC, PRAD) via GMQL/PyGMQL for the GRCh38 assembly, restricted to protein-coding and long non-coding genes. Compute zFPKM for each gene/sample pair as the z-score of log(FPKM) values following the Hart et al. normalization approach. Compute the mean zFPKM per gene within each condition (normal and cancer), and retain genes whose mean zFPKM is higher than −3.0 in BOTH conditions, discarding background/noisy genes. Write the retained genes and their zFPKM values to active_genes_zfpkm.csv. The rationale is that low-expression genes carry noise that corrupts downstream co-expression similarity estimation; on average about 20% of RNA genes pass this filter.

## Related tools

- **GMQL** (Query language used to retrieve the TCGA gene expression data and metadata from the TCGA repository) — http://gmql.eu
- **PyGMQL** (Python API used together with GMQL to fetch the TCGA gene expression datasets) — https://pygmql.readthedocs.io/en/latest/
- **Python** (Implements the preprocessing steps, including zFPKM computation and the mean zFPKM > −3.0 filtering (preprocessing.py)) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/preprocessing.py
- **Jupyter Notebook** (Pipeline.ipynb contains the complete pipeline, including this preprocessing/filtering step) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb

## Examples

```
pip install -r requirements.txt; jupyter notebook Pipeline.ipynb
```

## Evaluation signals

- All retained genes have mean zFPKM strictly higher than −3.0 in both the normal and the cancer condition
- The output file active_genes_zfpkm.csv exists and contains a zFPKM field per gene
- Retention rate is in the expected range: on average about 20% of RNA (protein-coding or long non-coding) genes pass this filter
- The preprocessing script runs successfully end-to-end within Pipeline.ipynb (requirements installed via pip install -r requirements.txt)

## Limitations

- Only genes expressed above the zFPKM −3.0 mean threshold in BOTH normal and cancer conditions are kept, so condition-specific expression (e.g. genes active only in tumor) is excluded from downstream networks
- The filter is applied only to protein-coding and long non-coding RNA genes; miRNAs follow a separate pre-extraction rate (81% on average)

## Evidence

- [methods] zFPKM threshold: "The selected RNA-Seq data have a mean value of normalized fragments per kilobase of transcript per million mapped reads (zFPKM) higher than −3.0"
- [results] pre-extraction rates: "On average, 20% of RNA (protein-coding or long non-coding) genes and 81% of miRNAs are extracted"
- [readme] data retrieval: [evidence span withheld — anchor status: external_doc]
- [readme] pipeline entry point: "The jupyter notebook [Pipeline.ipynb](https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb) contains the complete pipeline for the construction of the fused co-expression networks"
