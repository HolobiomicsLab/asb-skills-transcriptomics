---
name: droplet-scrnaseq-count-preprocessing-with-xcltk
description: Use when starting from raw 10x Genomics droplet scRNA-seq BAM/CRAM files
  of a tumor sample and you need RDR read-depth and BAF allelic AD/DP matrices to
  run XClone or similar allele-specific CNA inference.
when_to_use_negative:
- Input is already a processed gene expression or count matrix without allele-specific
  information — xcltk's BAF module requires BAM/CRAM alignments to genotype SNPs
- The downstream method is InferCNV or CopyKAT, which use expression-only inputs and
  do not consume xcltk's AD/DP allelic matrices
- Analysis targets bulk DNA-seq or single-cell DNA-seq (e.g. ACT/CHISEL workflows)
  rather than scRNA-seq read-depth/allelic signals
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0114
tools:
- name: xcltk
  role: Preprocessing toolkit that generates RDR, AD and DP count matrices from SAM/BAM/CRAM
    files for XClone
  repo: https://github.com/hxj5/xcltk
- name: XClone
  role: Downstream method consuming the xcltk-generated matrices to infer haplotype-specific
    CNAs
  repo: https://github.com/single-cell-genetics/XClone
- name: cellsnp-lite
  role: Original genotyping software that was re-designed into the tailored xcltk
    package
- name: xclone-data
  role: Repository of processed datasets (e.g. ATC2_scRNA, TNBC1_scRNA) and Jupyter
    notebook examples for reproducible results
  repo: https://github.com/Rongtingting/xclone-data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51026-0
    title: Robust analysis of allele-specific copy number alterations from scRNA-seq
      data with XClone
  source_content_hash: sha256:4376bfb412ab6e099eb2ba3a93f7e8a09de7acb69e007071dc5baaa2af33b1cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hxj5/xcltk
  tools:
  - CopyKAT
  - InferCNV
  - XClone
  - cellsnp-lite
  - scanpy
  - xclone-data
  - xcltk
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# droplet-scrnaseq-count-preprocessing-with-xcltk

## Summary

Preprocess droplet-based (10x Genomics) scRNA-seq data with the xcltk toolkit to generate the three count matrices XClone requires — total read depth (RDR signals) and allele-specific AD/DP (BAF signals) — from SAM/BAM/CRAM files, producing AnnData objects ready for downstream RDR/BAF CNA modeling.

## When to use

Use when starting from raw 10x Genomics droplet scRNA-seq BAM/CRAM files of a tumor sample and you need RDR read-depth and BAF allelic AD/DP matrices to run XClone or similar allele-specific CNA inference.

## When NOT to use

- Input is already a processed gene expression or count matrix without allele-specific information — xcltk's BAF module requires BAM/CRAM alignments to genotype SNPs
- The downstream method is InferCNV or CopyKAT, which use expression-only inputs and do not consume xcltk's AD/DP allelic matrices
- Analysis targets bulk DNA-seq or single-cell DNA-seq (e.g. ACT/CHISEL workflows) rather than scRNA-seq read-depth/allelic signals

## Inputs

- Droplet-based (10x Genomics) scRNA-seq alignment files (SAM/BAM/CRAM), e.g. ATC2 from GEO accession GSE148673
- Cell annotations (cell_anno_key) with normal-cell labels (ref_celltype)
- Genome feature annotations (e.g. hg38_genes, hg38_blocks) via xclone.pp.load_anno

## Outputs

- Total read depth count matrix (RDR signals) as AnnData
- Allele-specific AD and DP count matrices (BAF signals) as AnnData
- AnnData objects prepared via xclone.pp.xclonedata for xclone.model.run_RDR and run_BAF

## How to apply

Install xcltk (pip install -U xcltk, or from github.com/hxj5/xcltk for the development version) and run its RDR module and BAF module independently on the aligned 10x data. The xcltk preprocessing pipeline converts SAM/BAM/CRAM files into the total read depth matrix (RDR signals) and the allele-specific AD and DP matrices (BAF signals); it supports droplet-based platforms such as 10x Genomics and well-based platforms such as SMART-seq. Load the resulting matrices with xclone.pp.xclonedata (data_mode 'RDR' or 'BAF', genome_mode e.g. 'hg38_genes') to build AnnData objects, attaching cell annotations via cell_anno_key with ref_celltype identifying normal cells. Before model fitting, apply quality controls such as the RDR extra_preprocess cell-quality checks and libratio capping with cell filter limits. Output AnnData files then feed directly into xclone.model.run_RDR and run_BAF.

## Related tools

- **xcltk** (Preprocessing toolkit that generates RDR, AD and DP count matrices from SAM/BAM/CRAM files for XClone) — https://github.com/hxj5/xcltk
- **XClone** (Downstream method consuming the xcltk-generated matrices to infer haplotype-specific CNAs) — https://github.com/single-cell-genetics/XClone
- **cellsnp-lite** (Original genotyping software that was re-designed into the tailored xcltk package)
- **xclone-data** (Repository of processed datasets (e.g. ATC2_scRNA, TNBC1_scRNA) and Jupyter notebook examples for reproducible results) — https://github.com/Rongtingting/xclone-data

## Examples

```
pip install -U xcltk; RDR_Xdata = xclone.model.run_RDR(RDR_adata, config_file = xconfig)
```

## Evaluation signals

- Three matrices are produced: total read depth (RDR) plus allele-specific AD and DP; missing any one means preprocessing is incomplete
- AnnData objects load successfully with xclone.pp.xclonedata under the correct data_mode ('RDR' or 'BAF') and genome_mode (e.g. 'hg38_genes')
- Cell quality checks pass in RDR extra_preprocess, and libratio capping / cell filter limits do not flag excessive abnormal cells
- Downstream run_RDR and run_BAF run to completion and the combined haplotype-aware CNA calls recover known subclones (e.g. ATC2 gains on chr5, chr7, chr19 and loss on chr6p) with ARI ~0.998 vs inferCNV
- Environment matches the recommended setup, e.g. Python 3.9 with the xclone0.3.6Python3.9_env.yml conda specification

## Limitations

- Coverage of a single variant is extremely low in scRNA-seq; independent use of individual SNPs (as in HoneyBADGER/CaSpER) suffers high noise, which is why XClone's smoothing is applied after preprocessing
- Preprocessing requires access to aligned BAM/CRAM files, not just count matrices, limiting reuse of preprocessed public releases
- XClone is being updated frequently; version/environment control (Python 3.7 or >=3.9, recommend 3.9) is required for stable performance

## Evidence

- [readme] xcltk is a toolkit for XClone preprocessing.: [evidence span withheld — anchor status: external_doc]
- [readme] xcltk generates three matrices from BAM/CRAM: "It takes three matrices as input: the allele-specific *AD* and *DP* matrices
(BAF signals) and the *total read depth* matrix (RDR signals)."
- [readme] xcltk supports multiple platforms: "It supports data from multiple single-cell sequencing platforms, including 
droplet-based (e.g., 10x Genomics) and well-based (e.g., SMART-seq)
platforms."
- [methods] Article: two-step preprocessing workflow: "Step1: :ref:`Install xcltk <xcltk installation>` and run xctlkt :ref:`RDR module <xcltk RDR>` and :ref:`BAF module <xcltk BAF>`, independently."
- [intro] Article: cellsnp-lite re-designed into xcltk: "we re-designed our original genotyping software cellsnp-lite13 to a tailored package xcltk"
- [readme] README: install xcltk via pip: "pip install -U git+https://github.com/hxj5/xcltk"
- [methods] Article: RDR module invocation: "RDR_Xdata = xclone.model.run_RDR(RDR_adata,
                config_file = xconfig)"
