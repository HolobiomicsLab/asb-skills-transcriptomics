---
name: bisulfite-read-alignment
description: Use when single-cell RRBS (bisulfite) FASTQ reads must be aligned and
  per-cell SNVs called before building an n_cells x n_SNVs matrix for eeSNV inference
  with SSrGE.
when_to_use_negative:
- Input reads are standard (non-bisulfite-converted) scRNA-seq or DNA-seq — a bisulfite-aware
  aligner like Bismark is unnecessary and inappropriate; the SNV_calling pipeline
  alone suffices.
- SNVs are already provided as per-cell .vcf files — skip alignment and build the
  SNV/GE matrices directly from the existing snv_filtered.vcf files.
- The goal is DNA methylation-level analysis (methylation calls) rather than SNV/eeSNV
  extraction — this skill targets SNV inference from RRBS reads, not methylation quantification.
edam_operation: http://edamontology.org/operation_3185
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3173
tools:
- name: SSrGE docker pipeline
  role: Downloads GSE65364 SRA data, aligns reads, and runs per-cell SNV inference
    (process_snv) producing snv_filtered.vcf files
  repo: https://github.com/lanagarmire/SSrGE.git
- name: Bismark
  role: Bisulfite-aware aligner for the RRBS reads before SNV inference
- name: STAR
  role: Aligns expression reads with -read_length 100 -cell_type HUMAN to produce
    matrix_counts.txt gene counts
  repo: https://github.com/lanagarmire/SSrGE.git
- name: SSrGE
  role: Consumes the encoded SNV matrix and gene expression matrix to infer eeSNVs
    linked to expression
  repo: https://github.com/lanagarmire/SSrGE.git
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:77f3685a73f9f4507241935fd200a8079c941227868b2787b4498968c8de5994
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - Bismark
  - Numpy
  - Python
  - SSrGE
  - SSrGE docker pipeline
  - STAR
  - Scipy
  - scikit-learn
  - tabulate
  edam_operation: http://edamontology.org/operation_3185
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# bisulfite-read-alignment

## Summary

Align bisulfite-converted RRBS reads from coupled single-cell DNA methylation/RNA-seq data (e.g., the Hou HCC dataset, GSE65364) with a bisulfite-aware aligner such as Bismark, then infer per-cell SNVs to build the SNV matrices that SSrGE consumes as eeSNV features.

## When to use

Use when single-cell RRBS (bisulfite) FASTQ reads must be aligned and per-cell SNVs called before building an n_cells x n_SNVs matrix for eeSNV inference with SSrGE.

## When NOT to use

- Input reads are standard (non-bisulfite-converted) scRNA-seq or DNA-seq — a bisulfite-aware aligner like Bismark is unnecessary and inappropriate; the SNV_calling pipeline alone suffices.
- SNVs are already provided as per-cell .vcf files — skip alignment and build the SNV/GE matrices directly from the existing snv_filtered.vcf files.
- The goal is DNA methylation-level analysis (methylation calls) rather than SNV/eeSNV extraction — this skill targets SNV inference from RRBS reads, not methylation quantification.

## Inputs

- Bisulfite-converted single-cell RRBS FASTQ reads (e.g., GSE65364 SRA files, Hou dataset)
- Reference genome index (generated via generate_refgenome_index.py)
- Per-cell STAR gene expression counts (matrix_counts.txt)
- SSrGE docker container opoirion/ssrge

## Outputs

- Bisulfite-aligned BAM files per cell
- One snv_filtered.vcf per cell folder
- STAR matrix_counts.txt gene count files per cell
- Encoded n_cells x n_SNVs SNV matrix (0/0.5/1) ready for ssrge.fit(X, Y)

## How to apply

Download the SRA data with the SSrGE docker pipeline (docker run opoirion/ssrge download_soft_file/download_sra/extract_sra with a -project_name), then align the bisulfite-converted RRBS reads with a bisulfite-aware aligner such as Bismark; in parallel, align expression reads with STAR using -read_length 100 -cell_type HUMAN. Run SNV inference per cell with 'process_snv -cell_type HUMAN', producing one snv_filtered.vcf per cell folder alongside STAR matrix_counts.txt gene count files. Encode the resulting SNVs as a binary matrix X_c,n = 1 if SNV s is present in cell c (0.5 for heterozygous 0/1, 1 for homozygous 1/1 genotypes allowed), which is the feature matrix SSrGE fits against the gene expression matrix (optionally with a CNV matrix via ssrge.fit(X, Y, C)). Correct application is judged by whether each cell folder contains a well-formed snv_filtered.vcf and whether the eeSNV-space projection separates known cell groups (e.g., normal hepatocytes vs. HCC subtypes).

## Related tools

- **SSrGE docker pipeline** (Downloads GSE65364 SRA data, aligns reads, and runs per-cell SNV inference (process_snv) producing snv_filtered.vcf files) — https://github.com/lanagarmire/SSrGE.git
- **Bismark** (Bisulfite-aware aligner for the RRBS reads before SNV inference)
- **STAR** (Aligns expression reads with -read_length 100 -cell_type HUMAN to produce matrix_counts.txt gene counts) — https://github.com/lanagarmire/SSrGE.git
- **SSrGE** (Consumes the encoded SNV matrix and gene expression matrix to infer eeSNVs linked to expression) — https://github.com/lanagarmire/SSrGE.git

## Examples

```
docker run --rm -v $PATHDATA:/data/results/:Z opoirion/ssrge download_sra -project_name hcc_rrbs -max_nb_samples 2
```

## Evaluation signals

- Each cell folder contains a snv_filtered.vcf and a STAR matrix_counts.txt file, matching the expected directory layout (e.g., GSM2259781__SRX1999927__SRR3999457/snv_filtered.vcf).
- The encoded SNV matrix uses only values 0, 0.5 (heterozygous 0/1), or 1 (homozygous 1/1) per the article's encoding procedure.
- The simplified model built from top-ranked eeSNVs recovers ~92% eeSNV overlap.
- eeSNV-space projection clearly separates normal hepatocellular cells from HCC cells and highlights the two known HCC subtypes (Fig. 7).
- Subgroup analysis flags PRMT2 and SULF2 as significantly mutated in HCC cells (Fisher's exact test p < 0.05).

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads (RRBS provides reduced representation, not whole-genome coverage).
- In case of correlated features, LASSO may arbitrarily select one of several highly correlated SNV features corresponding to gene expression.
- Python 2.7.X must be used (pip2 install -r requirements.txt), and scikit-learn version 0.18 is required.

## Evidence

- [other] Align the RRBS reads (bisulfite-aware alignment, e.g. Bismark, or STAR for expression with -read_length 100 -cell_type HUMAN): [evidence span withheld — anchor status: not_found]
- [readme] SNV matrix encoding procedure: "if a given snv (s) is present into a given cell (c), then X_c,n = 1"
- [readme] Heterozygous/homozygous encoding allowance: "one can use X_c,n = 1 for a 1/1 genotype and 0.5 for a 0/1 genotype"
- [readme] SNV calling module producing per-cell vcf files: "SNV_calling: align reads/infer SNVs and infer gene expression matrices from .fastq files."
- [discussion] Incomplete DNA coverage caveat: "Nevertheless, SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads."
