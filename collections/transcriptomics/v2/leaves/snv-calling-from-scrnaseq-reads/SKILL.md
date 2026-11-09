---
name: snv-calling-from-scrnaseq-reads
description: Use when raw scRNA-seq reads (e.g. SRA files from a GEO study like the
  Kim dataset) need SNV genotyping per single cell, and no matched DNA-seq is available.
when_to_use_negative:
- Input is already an SNV matrix or VCF set — use the SSrGE modeling step directly
  instead of re-calling SNVs from reads
- Coupled DNA-seq data from the same single cells is available — DNA-seq-based variant
  calling gives more complete coverage than scRNA-seq reads
- Target variants lie outside exonic/covered regions — the pipeline may miss driver
  mutations due to incomplete DNA coverage of scRNA-seq reads
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: SSrGE
  role: Dockerized pipeline (image opoirion/ssrge) wrapping SRA download, STAR alignment,
    feature_counts, and GATK-based SNV calling from scRNA-seq reads
  repo: https://github.com/lanagarmire/SSrGE.git
- name: Python 2.7
  role: Runtime for the SSrGE package and its SNV_calling and download_ncbi_data modules
  repo: https://github.com/lanagarmire/SSrGE.git
- name: QuASAR
  role: Alternative R package referenced in the article's ecosystem for genotyping
    from sequencing reads via pileups; listed under 'other' in the article
  repo: https://github.com/piquelab/QuASAR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:70151e7bd1b1c8b193d8b547ce9ebe4d4be168ec6c729557f9a85b80c656e850
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - Python
  - Python 2.7
  - QuASAR
  - SSrGE
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# snv-calling-from-scrnaseq-reads

## Summary

A GATK-based SNV calling pipeline, distributed in the opoirion/ssrge docker image, that identifies single nucleotide variants directly from scRNA-seq raw reads (SRA/FASTQ) by aligning to hg19 with STAR and calling SNVs from the aligned BAMs, benchmarked against artificially seeded ground-truth variants.

## When to use

Use when raw scRNA-seq reads (e.g. SRA files from a GEO study like the Kim dataset) need SNV genotyping per single cell, and no matched DNA-seq is available.

## When NOT to use

- Input is already an SNV matrix or VCF set — use the SSrGE modeling step directly instead of re-calling SNVs from reads
- Coupled DNA-seq data from the same single cells is available — DNA-seq-based variant calling gives more complete coverage than scRNA-seq reads
- Target variants lie outside exonic/covered regions — the pipeline may miss driver mutations due to incomplete DNA coverage of scRNA-seq reads

## Inputs

- Raw scRNA-seq sequencing data in SRA format for individual single cells (e.g. Kim dataset, 20 single cells)
- Reference genome (hg19 human reference, FASTA) with read_length 100 and cell_type HUMAN for the STAR index
- Optionally: per-cell snv_filtered.vcf and matrix_counts.txt files for downstream SNV/GE matrix construction

## Outputs

- Per-cell aligned BAM files (STAR against hg19)
- Gene count matrices (matrix_counts.txt from feature_counts)
- Per-cell SNV calls in VCF format (snv_filtered.vcf)
- TPR and FPR per read-depth bin when validated against seeded ground-truth SNVs

## How to apply

Run the SSrGE docker image opoirion/ssrge: download the SOFT file and SRA files for the single cells (download_soft_file, download_sra), extract them (extract_sra), then build a STAR index for the hg19 reference with read_length 100 and cell_type HUMAN (generate_refgenome_index / STAR indexing step). Align each cell's reads to hg19 with process_star (read_length 100, HUMAN), produce gene count matrices with feature_counts, and call SNVs with process_snv (cell_type HUMAN). For validation, seed each BAM with ~50,000 randomly placed exonic SNVs as ground truth and compute true/false positive rates per read-depth bin. Accept the run if TPR exceeds 50% at read depth 4, reaches ~68% at depth >6, and the median FPR stays below 0.05 at depth >6, matching the reported Fig. 1a, b results.

## Related tools

- **SSrGE** (Dockerized pipeline (image opoirion/ssrge) wrapping SRA download, STAR alignment, feature_counts, and GATK-based SNV calling from scRNA-seq reads) — https://github.com/lanagarmire/SSrGE.git
- **Python 2.7** (Runtime for the SSrGE package and its SNV_calling and download_ncbi_data modules) — https://github.com/lanagarmire/SSrGE.git
- **QuASAR** (Alternative R package referenced in the article's ecosystem for genotyping from sequencing reads via pileups; listed under 'other' in the article) — https://github.com/piquelab/QuASAR

## Examples

```
docker run --rm opoirion/ssrge download_sra -h  # then extract_sra, process_star (read_length 100, HUMAN) and process_snv (cell_type HUMAN) in the same image
```

## Evaluation signals

- TPR of recovered seeded SNVs exceeds 50% at read depth 4
- TPR reaches ~68% aggregated at read depth >6 (Fig. 1a, b reproduction)
- False positive rate is consistently <0.1 with median below 0.05 at depth >6
- STAR alignment ran with the expected parameters (hg19, read_length 100, cell_type HUMAN) and feature_counts produced per-cell matrix_counts.txt files
- Each cell folder contains a distinct snv_filtered.vcf after process_snv

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads
- SNV calling accuracy is depth-dependent: TPR is only ~50% at depth 4, so low-depth cells retain many uncalled true variants
- FPR is low but nonzero; LASSO-based downstream filtering is needed to minimize spurious eeSNVs

## Evidence

- [methods] docker benchmark workflow: "docker run --rm opoirion/ssrge download_soft_file -h"
- [methods] docker download_sra usage: "docker run --rm opoirion/ssrge download_sra -h"
- [methods] docker extract_sra usage: "docker run --rm opoirion/ssrge extract_sra -h"
- [readme] SSrGE SNV calling module: "[SNV_calling](https://github.com/lanagarmire/SSrGE/blob/master/README_snv_calling.md): align reads/infer SNVs and infer gene expression matrices from .fastq files."
- [readme] docker image with download, alignment and SNV calling: "we compiled the download, alignment, and SNV calling pipelines into a docker container: opoirion/ssrge"
- [results] depth-dependent TPR/FPR benchmark: [evidence span withheld — anchor status: not_found]
- [discussion] LASSO reduces spurious SNVs: "LASSO penalization works as a feature selection method and minimizes the spurious SNVs (false positive) from the filtered set of eeSNVs."
- [discussion] coverage limitation: "Nevertheless, SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads."
