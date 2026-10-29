---
name: rna-seq-expression-data-preprocessing
description: Use when raw TCGA RNA-Seq gene expression matrices must be filtered and
  normalized before computing gene co-expression networks, and only protein-coding/lncRNA
  genes with sufficient expression (mean zFPKM > -3.0) should be kept.
when_to_use_negative:
- Input data is already a filtered expression matrix or a precomputed co-expression
  network rather than raw TCGA RNA-Seq expression data
- The target gene set includes miRNAs or non-coding RNAs other than protein-coding
  and long non-coding RNA genes, which are excluded by this preprocessing filter
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: GMQL
  role: Query language used to retrieve TCGA RNA-Seq gene expression data and metadata
  repo: http://gmql.eu
- name: PyGMQL
  role: Python interface through which GMQL-retrieved gene expression data are loaded
    into the preprocessing pipeline
  repo: https://pygmql.readthedocs.io/en/latest/
- name: Python
  role: Language implementing preprocessing.py, the script containing this preprocessing
    step
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Pipeline.ipynb notebook that executes the complete pipeline including preprocessing
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:37ddcf0e289c6abc07ae56c817296efaad54be03070c67b80edb4feb56119113
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GMQL
  - Jupyter Notebook
  - PyGMQL
  - Python
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# rna-seq-expression-data-preprocessing

## Summary

Pre-processes TCGA RNA-Seq gene expression data using GMQL/PyGMQL, retaining protein-coding and long non-coding RNA genes whose mean zFPKM exceeds -3.0 in both normal and cancer conditions, as the first step of building fused co-expression networks for KIRC, LIHC and PRAD.

## When to use

Use when raw TCGA RNA-Seq gene expression matrices must be filtered and normalized before computing gene co-expression networks, and only protein-coding/lncRNA genes with sufficient expression (mean zFPKM > -3.0) should be kept.

## When NOT to use

- Input data is already a filtered expression matrix or a precomputed co-expression network rather than raw TCGA RNA-Seq expression data
- The target gene set includes miRNAs or non-coding RNAs other than protein-coding and long non-coding RNA genes, which are excluded by this preprocessing filter

## Inputs

- TCGA RNA-Seq gene expression data retrieved via GMQL/PyGMQL (Datasets folder)
- Gene annotation mapping gene symbols to gene types (protein-coding vs long non-coding RNA), e.g. matrix_GTypeGName.xlsx

## Outputs

- Filtered, pre-processed gene expression matrices for normal and cancer conditions per cancer type (Datasets_created folder)
- Input matrices for subsequent Euclidean-distance and Pearson-correlation co-expression network computation

## How to apply

Extract TCGA RNA-Seq gene expression data through GMQL queries executed from Python via PyGMQL. Select only RNA genes of type protein-coding or long non-coding RNA, and keep genes whose mean value of normalized fragments per kilobase of transcript per million mapped reads (zFPKM) is higher than -3.0 in both the normal and the cancer condition of each cancer type. The article reports that on average 20% of RNA genes and 81% of miRNAs are extracted in this step, so a filtered matrix of roughly a fifth of input genes is expected. The implementation lives in preprocessing.py in the DEIB-GECO/GeneNetFusion repository and is also executed as part of the Pipeline.ipynb Jupyter notebook, which produces the Datasets_created folder used as input for co-expression matrix creation (creation_matrices.py). Output must be condition-specific expression matrices (normal and cancer per cancer type) ready for Euclidean-distance and Pearson-correlation network construction.

## Related tools

- **GMQL** (Query language used to retrieve TCGA RNA-Seq gene expression data and metadata) — http://gmql.eu
- **PyGMQL** (Python interface through which GMQL-retrieved gene expression data are loaded into the preprocessing pipeline) — https://pygmql.readthedocs.io/en/latest/
- **Python** (Language implementing preprocessing.py, the script containing this preprocessing step) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Pipeline.ipynb notebook that executes the complete pipeline including preprocessing) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb

## Examples

```
pip install -r requirements.txt; jupyter notebook Pipeline.ipynb
```

## Evaluation signals

- Filtered gene set contains only protein-coding and long non-coding RNA gene types, checked against the symbol-to-type mapping (matrix_GTypeGName.xlsx)
- Every retained gene has mean zFPKM strictly higher than -3.0 in both the normal and the cancer condition
- Expected scale: on average about 20% of RNA (protein-coding or long non-coding) genes are extracted relative to the input TCGA gene universe
- Outputs are formatted as condition-specific expression matrices consumable by creation_matrices.py for Euclidean-distance and Pearson-correlation network computation
- Running preprocessing.py or Pipeline.ipynb completes without errors and populates the Datasets_created folder with the expected files

## Limitations

- The filter excludes miRNAs from the gene co-expression pipeline even though the article reports 81% of miRNAs are extracted, so miRNA analyses need a separate path
- The zFPKM > -3.0 cutoff removes lowly expressed genes; genes relevant to a cancer type but weakly expressed in both conditions are lost before network construction
- Repository documentation only ships data, scripts and results for the LIHC tumor analysis, so KIRC and PRAD preprocessing outputs must be regenerated by the user

## Evidence

- [methods] The selected RNA-Seq data have a mean value of normalized fragments per kilobase of transcript per million mapped reads (zFPKM) higher than −3.0: "The selected RNA-Seq data have a mean value of normalized fragments per kilobase of transcript per million mapped reads (zFPKM) higher than −3.0"
- [results] On average, 20% of RNA (protein-coding or long non-coding) genes and 81% of miRNAs are extracted: "On average, 20% of RNA (protein-coding or long non-coding) genes and 81% of miRNAs are extracted"
- [readme] gene expression data retrieved through GMQL and PyGMQL: "that contain the TCGA gene expression data retrieved through [GMQL](http://gmql.eu) and [PyGMQL](https://pygmql.readthedocs.io/en/latest/)"
- [full_text] Extract and pre-process TCGA RNA-Seq gene expression data using GMQL/PyGMQL: [evidence span withheld — anchor status: not_found]
