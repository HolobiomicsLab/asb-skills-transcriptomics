---
name: expression-based-regulome-prediction
description: Use when you have RNA-seq (FPKM matrix) or exon array gene expression
  for a sample and need genome-scale chromatin accessibility predictions, e.g., to
  pseudo-replicate DNase-seq or convert GEO expression samples into regulomes.
when_to_use_negative:
- You need to discover new DHS locations not represented in the model's ~1M pre-specified
  loci — BIRD will not discover new locations of DHSs
- Input is raw reads rather than gene expression (process with Tophat/Cufflinks or
  HISAT2/StringTie first to obtain FPKM)
- Data are from non-human species or platforms lacking a prebuilt BIRD model
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: BIRD
  role: Core big data regression software that predicts DNase I hypersensitivity genome-wide
    from gene expression; installed from source with make
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD_predict
  role: Prediction command invoked with a model binary (-b), an FPKM matrix (-i) and
    output (-o); performs ensembl gene id matching, quantile normalization, and outputs
    log2(x+1) predictions or WIG files
  repo: https://github.com/WeiqiangZhou/BIRD
- name: GeneBASE
  role: Processes raw exon array CEL files into the gene expression input file accepted
    by BIRD for exon array prediction
  repo: https://github.com/WeiqiangZhou/BIRD
- name: Tophat
  role: Used with Cufflinks to obtain gene expression (FPKM) for the BIRD input sample
- name: Cufflinks
  role: Used with Tophat to obtain gene expression (FPKM) for the BIRD input sample
- name: R
  role: Required installation so the Rscript command is executable (BIRD scripts depend
    on it); used downstream for loading BIRD-data rds training datasets
  repo: https://github.com/WeiqiangZhou/BIRD
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:38280d885860c725bfa6464e63fc6211f70be7064bb0ac25dcfe6e2683dfd294
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WeiqiangZhou/BIRD
  tools:
  - BIRD
  - BIRD_predict
  - Cufflinks
  - GeneBASE
  - R
  - Tophat
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# expression-based-regulome-prediction

## Summary

Predict genome-wide DNase I hypersensitivity (chromatin accessibility) across ~1M genomic loci from a biological sample's transcriptome using BIRD, a big data regression model trained on ENCODE data, enabling regulome mapping for samples with only gene expression data.

## When to use

Use when you have RNA-seq (FPKM matrix) or exon array gene expression for a sample and need genome-scale chromatin accessibility predictions, e.g., to pseudo-replicate DNase-seq or convert GEO expression samples into regulomes.

## When NOT to use

- You need to discover new DHS locations not represented in the model's ~1M pre-specified loci — BIRD will not discover new locations of DHSs
- Input is raw reads rather than gene expression (process with Tophat/Cufflinks or HISAT2/StringTie first to obtain FPKM)
- Data are from non-human species or platforms lacking a prebuilt BIRD model

## Inputs

- FPKM gene expression matrix with gene ensembl ids in the first column (FPKM_data_matrix.txt)
- GeneBASE gene expression output file for exon array data (e.g., Exon_K562_lab.txt)
- Prebuilt BIRD model binary (e.g., human_hg38_model.bin, Exonarray_model_file.bin)

## Outputs

- Data matrix of predicted DNase I hypersensitivity values per locus per sample, log2(x+1) transformed with upper bound 14 (output_file.txt)
- Matched input matrix written as input_file.match.txt (e.g., FPKM_data_matrix.txt.match.txt)
- Per-sample WIG files (with -w, or -wt for log-transformed) for UCSC genome browser / IGV visualization

## How to apply

Compile BIRD from source (unzip BIRD_v1.1.1.zip; make) and download a prebuilt model matched to your genome and platform (e.g., human_hg38_model.bin for hg38 RNA-seq FPKM, or Exonarray_model_file.bin for exon array). Prepare an FPKM data matrix with gene ensembl ids in the first column and expression values per sample in other columns; BIRD_predict matches ensembl ids automatically (ignoring version suffixes unless -e is given). Run BIRD_predict with the model via -b to produce a matrix of predicted values in log2(x+1) scale bounded from 0 to 14 (default upper bound changeable with -u), or add -w/-wt for per-sample WIG files for UCSC/IGV visualization. For exon array inputs, first process raw CEL files with GeneBASE. Note that genomic coordinates are hg19 for earlier models/hg38 for the newest model, so downstream tracks must be aligned on the correct genome build's bin coordinates.

## Related tools

- **BIRD** (Core big data regression software that predicts DNase I hypersensitivity genome-wide from gene expression; installed from source with make) — https://github.com/WeiqiangZhou/BIRD
- **BIRD_predict** (Prediction command invoked with a model binary (-b), an FPKM matrix (-i) and output (-o); performs ensembl gene id matching, quantile normalization, and outputs log2(x+1) predictions or WIG files) — https://github.com/WeiqiangZhou/BIRD
- **GeneBASE** (Processes raw exon array CEL files into the gene expression input file accepted by BIRD for exon array prediction) — https://github.com/WeiqiangZhou/BIRD
- **Tophat** (Used with Cufflinks to obtain gene expression (FPKM) for the BIRD input sample)
- **Cufflinks** (Used with Tophat to obtain gene expression (FPKM) for the BIRD input sample)
- **R** (Required installation so the Rscript command is executable (BIRD scripts depend on it); used downstream for loading BIRD-data rds training datasets) — https://github.com/WeiqiangZhou/BIRD

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt
```

## Evaluation signals

- Output matrix values are log2(x+1) transformed and bounded in [0, 14] (unless -u changes the upper bound)
- A matched input file (e.g., FPKM_data_matrix.txt.match.txt) is produced, confirming ensembl id matching between model genes and the input matrix
- WIG tracks (with -w) render sensible DNase-seq-like signal when added as custom tracks in the UCSC genome browser or IGV
- Predicted DH correlates with true DNase-seq signal, e.g., ~0.82 Pearson correlation against a held-out replicate when used as a pseudo-replicate baseline in test cell types
- Model and genome build are consistent: hg38 predictions use human_hg38_model.bin; hg19 tracks align on hg19 coordinates

## Limitations

- BIRD will not discover new locations of DHSs when applied to a new gene expression sample — predictions are limited to loci in the training model
- Genomic information in earlier versions of BIRD is based on human genome hg19; newer models target hg38 — coordinate builds must not be mixed
- Whether gene expression can be similarly used to predict other functional genomic data types remains an open question; extending BIRD to other expression platforms is future work

## Evidence

- [methods] BIRD_predict is run to produce log2(x+1) transformed predictions bounded at 14: "path_to_BIRD/BIRD_predict -b path_to_model/human_hg38_model.bin -i FPKM_data_matrix.txt -o output_file.txt"
- [readme] The standard output matrix is log2(x+1) transformed with an upper bound of 14: "Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed)."
- [readme] Predicted values are bounded from 0 to 14 with -u to change the upper bound: "The predicted values are now bounded from 0 to 14. Users can use the -u option to change the upper bound when using their own prediction model."
- [readme] Ensembl gene id matching is built into BIRD_predict, ignoring version suffixes by default: "BIRD will first match the ensembl id of the genes used for building the prediction model with the ids in the input data matrix."
- [abstract] Gene expression predicts DH with information contained in the whole transcriptome: "gene expression predicts DH, and information useful for prediction is contained in the whole transcriptome rather than limited to a regulatory element's neighboring genes"
- [discussion] A key limitation: BIRD cannot discover new DHS locations for new expression samples: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
