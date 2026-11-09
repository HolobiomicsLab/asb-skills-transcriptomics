---
name: single-cell-snv-calling
description: Use when starting from raw scRNA-seq reads (GEO SRA/FASTQ) and you need
  per-cell SNV calls and gene count matrices as SSrGE inputs, e.g. to build eeSNV
  features for cancer subpopulation identification.
when_to_use_negative:
- You already have per-cell VCFs or an SNV matrix and want to fit SSrGE LASSO models
  or rank eeSNVs — this skill only produces the SNV/count inputs, not the modeling.
- The goal is exhaustive variant discovery from matched DNA-seq; the paper warns SSrGE
  may miss some driver mutations due to incomplete DNA coverage of scRNA-seq reads.
- The data are not single-cell RNA-seq reads, or library read length/cell type differ
  from the STAR index parameters (read_length 100, cell_type HUMAN) the index was
  built with.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: SSrGE (docker image opoirion/ssrge)
  role: Containerized pipeline for download/alignment/SNV calling; process_snv infers
    per-cell SNVs written as snv_filtered.vcf
  repo: https://github.com/lanagarmire/SSrGE.git
- name: GATK
  role: SNV calling guidelines followed when identifying SNVs from scRNA-seq raw reads
- name: STAR
  role: Aligner used via 'star_index' and 'process_star' docker commands (read_length
    100, cell_type HUMAN)
- name: featureCounts
  role: Generates per-sample gene count matrices (matrix_counts.txt) consumed together
    with the per-cell VCFs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:35bca3418524764e9eeac66eda69746be13509df14b2cbd23896c8c22427868e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - GATK
  - Python
  - SSrGE (docker image opoirion/ssrge)
  - STAR
  - featureCounts
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# single-cell-snv-calling

## Summary

Dockerized SSrGE pipeline (image opoirion/ssrge) that infers single-nucleotide variants directly from scRNA-seq raw reads: SRA download/extraction, STAR alignment, featureCounts gene matrices, then GATK-guideline SNV calling, emitting one snv_filtered.vcf per cell alongside a matrix_counts.txt gene-expression file.

## When to use

Use when starting from raw scRNA-seq reads (GEO SRA/FASTQ) and you need per-cell SNV calls and gene count matrices as SSrGE inputs, e.g. to build eeSNV features for cancer subpopulation identification.

## When NOT to use

- You already have per-cell VCFs or an SNV matrix and want to fit SSrGE LASSO models or rank eeSNVs — this skill only produces the SNV/count inputs, not the modeling.
- The goal is exhaustive variant discovery from matched DNA-seq; the paper warns SSrGE may miss some driver mutations due to incomplete DNA coverage of scRNA-seq reads.
- The data are not single-cell RNA-seq reads, or library read length/cell type differ from the STAR index parameters (read_length 100, cell_type HUMAN) the index was built with.

## Inputs

- GEO accession of a scRNA-seq study (e.g. the Kim dataset) with SRA reads
- .sra files (extracted to FASTQ)
- Reference genome GTF for building the GTF/STAR index
- Parameters: read_length 100, cell_type HUMAN, -project_name

## Outputs

- One snv_filtered.vcf per cell, in a uniquely named per-cell folder
- matrix_counts.txt gene count matrix per cell (plus matrix_counts.txt.summary)
- STAR alignment output and, optionally, removed .sra files after rm_sra

## How to apply

Pull the opoirion/ssrge docker image and download the GEO soft file and SRA reads (download_soft_file, download_sra), then extract with extract_sra. Generate the reference genome GTF index with 'python2 ./garmire_SSrGE/generate_refgenome_index.py', build a STAR index (read_length 100, cell_type HUMAN), and align reads with process_star using the same parameters. Produce per-sample gene count matrices with feature_counts -project_name <project>, then call SNVs per cell with process_snv -project_name <project> -cell_type HUMAN, following the SNV guideline of GATK. Validate that every per-cell folder (e.g. GSM2259781__SRX1999927__SRR3999457) contains a non-empty snv_filtered.vcf and a matrix_counts.txt whose header is gene_name/chromosomes/starting position/ending position/additional columns/gene expression, and that no .sra files remain if rm_sra was run. The rationale is to obtain SNV-based genetic information from scRNA-seq in addition to gene expression, since GE features alone are noisy for subpopulation characterization.

## Related tools

- **SSrGE (docker image opoirion/ssrge)** (Containerized pipeline for download/alignment/SNV calling; process_snv infers per-cell SNVs written as snv_filtered.vcf) — https://github.com/lanagarmire/SSrGE.git
- **GATK** (SNV calling guidelines followed when identifying SNVs from scRNA-seq raw reads)
- **STAR** (Aligner used via 'star_index' and 'process_star' docker commands (read_length 100, cell_type HUMAN))
- **featureCounts** (Generates per-sample gene count matrices (matrix_counts.txt) consumed together with the per-cell VCFs)

## Examples

```
docker run --rm -v $PATHDATA:/data/results/:Z opoirion/ssrge process_snv -project_name <project> -cell_type HUMAN
```

## Evaluation signals

- Every per-cell directory contains a non-empty snv_filtered.vcf and a matrix_counts.txt.
- matrix_counts.txt header follows the format gene_name/chromosomes/starting position/ending position/additional columns/gene expression.
- Each cell has a uniquely named folder (e.g. GSM2259781__SRX1999927__SRR3999457) with exactly one distinct .vcf file.
- No SRA files remain in the project directory if rm_sra was executed.
- The STAR index and process_star alignment were run with matching read_length (100) and cell_type (HUMAN) parameters.

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads.
- The docker route needs docker, possible root access, and 13.8 GB of free memory for the opoirion/ssrge image.
- Local package steps (e.g. generate_refgenome_index.py) require Python 2.7 with Numpy, Scipy, Scikit-learn 0.18, and tabulate.

## Evidence

- [intro] Pipeline objective: call SNVs straight from scRNA-seq raw reads: "Here we first built a computational pipeline to identify SNVs from scRNA-seq raw reads directly."
- [readme] Per-cell VCF layout expected by SSrGE: "Each cell must have a distinct .vcf file with a unique name (e.g. snv_filtered.vcf) inside a unique folder, specific of the cell"
- [readme] The SNV_calling module converts FASTQ to SNVs and gene expression: "align reads/infer SNVs and infer gene expression matrices from .fastq files."
- [methods] Reference GTF index must precede matrix generation: "python2 ./garmire_SSrGE/generate_refgenome_index.py"
- [methods] SRA retrieval step inside the dockerized workflow: "docker run --rm opoirion/ssrge download_soft_file -h"
