---
name: scrnaseq-read-alignment-star
description: Use when raw scRNA-seq reads (.sra/.fastq, e.g., from the Kim dataset)
  must be mapped to HUMAN reference (read_length 100) to produce BAMs feeding featureCounts
  and GATK-based SNV calling.
when_to_use_negative:
- Input is already an aligned BAM or a per-cell snv_filtered.vcf / matrix_counts.txt
  — skip directly to feature_counts or process_snv
- Data is bulk RNA-seq or coupled single-cell DNA-seq; the pipeline here targets scRNA-seq
  raw reads, and SSrGE notes it 'may miss some driver mutations due to the incomplete
  DNA coverage of scRNA-seq reads'
- Reads whose length differs substantially from the indexed read_length (100) — rebuild
  the STAR index with the matching read_length first
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: SSrGE
  role: Packages the download, STAR indexing and STAR alignment pipeline in the opoirion/ssrge
    docker container (star_index, process_star commands)
  repo: https://github.com/lanagarmire/SSrGE.git
- name: Python 2.7
  role: Runs the SSrGE package and generate_refgenome_index.py to prepare the GTF
    reference index used for downstream count matrices
  repo: https://github.com/lanagarmire/SSrGE.git
- name: opoirion/ssrge
  role: Docker image invoked for star_index and process_star with -v $PATHDATA:/data/results/:Z,
    read_length 100, cell_type HUMAN
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:2bb8491fe5f909c6dc678488eda8c43c022b1daf8c4316931b3f265e47ca4ebd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - Python
  - Python 2.7
  - SSrGE
  - opoirion/ssrge
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# scrnaseq-read-alignment-star

## Summary

Align single-cell RNA-seq FASTQ/SRA reads to a reference genome (e.g., hg19) using STAR as implemented in the SSrGE docker pipeline (opoirion/ssrge), as the step between SRA extraction and per-cell gene count matrix / SNV calling.

## When to use

Use when raw scRNA-seq reads (.sra/.fastq, e.g., from the Kim dataset) must be mapped to HUMAN reference (read_length 100) to produce BAMs feeding featureCounts and GATK-based SNV calling.

## When NOT to use

- Input is already an aligned BAM or a per-cell snv_filtered.vcf / matrix_counts.txt — skip directly to feature_counts or process_snv
- Data is bulk RNA-seq or coupled single-cell DNA-seq; the pipeline here targets scRNA-seq raw reads, and SSrGE notes it 'may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads'
- Reads whose length differs substantially from the indexed read_length (100) — rebuild the STAR index with the matching read_length first

## Inputs

- Extracted scRNA-seq FASTQ files from SRA (e.g., Kim dataset single-cell reads)
- STAR genome index built for the human reference (cell_type HUMAN, read_length 100)

## Outputs

- Per-cell aligned BAM files ready for feature_counts and process_snv
- Downstream per-cell matrix_counts.txt gene count matrices (via feature_counts)
- Per-cell snv_filtered.vcf files (via process_snv, GATK-based SNV calling)

## How to apply

After downloading GEO soft files and SRA reads and extracting them (download_soft_file, download_sra, extract_sra on the opoirion/ssrge image), first build a STAR index with 'docker run --rm -v $PATHDATA:/data/results/:Z opoirion/ssrge star_index', setting read_length 100 and cell_type HUMAN. Then align the extracted single-cell reads with 'docker run --rm -v $PATHDATA:/data/results/:Z opoirion/ssrge process_star' using the same read_length and cell_type so the index matches the read length. Alignment must precede feature_counts (per-sample gene count matrices, matrix_counts.txt) and process_snv (per-cell snv_filtered.vcf via the GATK SNV guideline), since both consume the aligned reads. Optionally run rm_sra afterwards to remove .sra files; the pipeline was validated in simulation by seeding 50,000 exonic SNVs into BAMs and achieving >50% TPR at depth 4 and ~68% at depth >6 with FPR <0.1.

## Related tools

- **SSrGE** (Packages the download, STAR indexing and STAR alignment pipeline in the opoirion/ssrge docker container (star_index, process_star commands)) — https://github.com/lanagarmire/SSrGE.git
- **Python 2.7** (Runs the SSrGE package and generate_refgenome_index.py to prepare the GTF reference index used for downstream count matrices) — https://github.com/lanagarmire/SSrGE.git
- **opoirion/ssrge** (Docker image invoked for star_index and process_star with -v $PATHDATA:/data/results/:Z, read_length 100, cell_type HUMAN)

## Examples

```
docker run --rm -v $PATHDATA:/data/results/:Z opoirion/ssrge process_star -read_length 100 -cell_type HUMAN
```

## Evaluation signals

- Every cell directory contains aligned-read-derived outputs: a non-empty snv_filtered.vcf and a matrix_counts.txt whose header follows gene_name/chromosomes/starting position/ending position/additional columns/gene expression
- STAR index was built with the same read_length (100) and cell_type (HUMAN) parameters used for process_star alignment
- If rm_sra was run, no .sra files remain in the working directory
- In benchmark mode, seeding 50,000 randomly placed exonic SNVs into aligned BAMs yields >50% TPR at read depth 4 and ~68% TPR at depth >6, with median FPR <0.05 at depth >6

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads
- Alignment accuracy at low read depth limits SNV recovery: true positive rate is only ~50% at read depth 4, rising to 68% only above depth 6
- Docker image requires ~13.8 GB of free memory and possible root access

## Evidence

- [methods] STAR index built with docker star_index command: [evidence span withheld — anchor status: not_found]
- [intro] SNV calling from raw reads: "Here we first built a computational pipeline to identify SNVs from scRNA-seq raw reads directly."
- [readme] SNV calling module README: "[SNV_calling](https://github.com/lanagarmire/SSrGE/blob/master/README_snv_calling.md): align reads/infer SNVs and infer gene expression matrices from .fastq files."
- [discussion] Alignment depth limitation: "Nevertheless, SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads."
