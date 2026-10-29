---
name: viral-reference-genome-preparation
description: Use when running scPathoQuant on 10x cellranger output and a pathogen
  (e.g. HIV, SARS-CoV-2, metapneumovirus) reference genome must be packaged into the
  -p2genome folder with clean FASTA headers.
when_to_use_negative:
- The analysis target is not a 10x scRNAseq pathogen-quantification run (e.g. bulk
  RNA-seq or metagenomic assembly) — this folder format is specific to scPathoQuant's
  -p2genome argument.
- 'A host genome reference is needed: scPathoQuant expects the host alignment to already
  be done by cellranger count; the -p2genome folder must contain only pathogen sequences,
  not the host genome.'
- Per-gene pathogen counts are not required and no GTF is available — a FASTA-only
  folder still works, so blocking on GTF creation is unnecessary.
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3170
tools:
- name: scPathoQuant
  role: Consumes the prepared -p2genome folder; uses the FASTA header to quantify
    total pathogen reads per cell
  repo: https://github.com/galelab/scPathoQuant
- name: cellranger
  role: Must have already run count on the host genome so unaligned reads can be re-mapped
    to the prepared pathogen reference
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:24958a793dfec26610c2671e5591c134c67b10d0886ab6a6c56ec0dc778d4490
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - Python
  - bbmap
  - bowtie2
  - cellranger
  - scPathoQuant
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# viral-reference-genome-preparation

## Summary

Assembles the pathogen reference folder that scPathoQuant consumes via -p2genome: a simplified-header FASTA (and optionally a GTF) per viral genome, capped at 2 files, since the FASTA header is the key under which total pathogen reads are quantified.

## When to use

Use when running scPathoQuant on 10x cellranger output and a pathogen (e.g. HIV, SARS-CoV-2, metapneumovirus) reference genome must be packaged into the -p2genome folder with clean FASTA headers.

## When NOT to use

- The analysis target is not a 10x scRNAseq pathogen-quantification run (e.g. bulk RNA-seq or metagenomic assembly) — this folder format is specific to scPathoQuant's -p2genome argument.
- A host genome reference is needed: scPathoQuant expects the host alignment to already be done by cellranger count; the -p2genome folder must contain only pathogen sequences, not the host genome.
- Per-gene pathogen counts are not required and no GTF is available — a FASTA-only folder still works, so blocking on GTF creation is unnecessary.

## Inputs

- Pathogen reference genome FASTA (e.g. SARS-CoV-2 GCF_009858895.2, HIV genome from testSets genomefiles/)
- Optional pathogen GTF annotation file
- List of viruses expected in the sample (single or co-infection)

## Outputs

- -p2genome reference folder with ≤2 files: simplified-header pathogen FASTA and optional pathogen GTF
- Multi-virus reference folder (e.g. SARS-CoV-2 + metapneumovirus) for co-infection runs

## How to apply

Create the -p2genome folder containing at most 2 files: the pathogen reference FASTA and, if per-gene quantification is desired, a matching pathogen GTF (the GTF is optional; FASTA-only runs still produce total per-cell pathogen read counts but skip pathogen gene counts). Simplify each FASTA header (e.g. '>HIV_virus') because scPathoQuant uses the header verbatim to quantify the number of total pathogen reads. For co-infection analysis, combine genomes for up to the desired viruses (e.g. SARS-CoV-2 from NCBI GCF_009858895.2 plus a metapneumovirus genome) in one folder so a single scPathoQuant run can differentiate multiple viruses in one sample. Obtain the SARS-CoV-2 reference genome and GTF from the NCBI datasets page GCF_009858895.2, or the HIV genome files from genomefiles/ in the galelab/Whitmore_scPathoQuant_testSets repository.

## Related tools

- **scPathoQuant** (Consumes the prepared -p2genome folder; uses the FASTA header to quantify total pathogen reads per cell) — https://github.com/galelab/scPathoQuant
- **cellranger** (Must have already run count on the host genome so unaligned reads can be re-mapped to the prepared pathogen reference)

## Examples

```
mkdir -p pathogen_ref && cp SARS-CoV-2.fa metapneumovirus.fa pathogen_ref/ && scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome pathogen_ref
```

## Evaluation signals

- The -p2genome folder contains at most 2 files: one FASTA and optionally one GTF, per the README's -p2genome description.
- Each FASTA header is simplified (e.g. '>HIV_virus' style), since complicated headers propagate into pathogen read quantification.
- For the co-infection test set, the folder contains both the SARS-CoV-2 reference (NCBI GCF_009858895.2) and a metapneumovirus genome, matching the Rosin_et_al_testSet genomefiles/ layout.
- The subsequent scPathoQuant run completes and reports pathogen counts under the expected simplified header name in pathogen_al_counts.csv.

## Limitations

- The FASTA header is used directly for quantifying total pathogen reads, so a complicated header will propagate into downstream count labels; it must be simplified beforehand.
- Per-gene (pathogen UMI gene) counts are only produced when a pathogen GTF is provided; a FASTA-only reference yields total-read counts only.
- The GTF is not mandatory, so gene-level output will silently be absent (pathogen_genes_al_sort_counts.sam not produced) if the GTF is omitted.

## Evidence

- [readme] README -p2genome description: "in this folder should be at most 2 files 1) the fasta file with the pathogen reference genome sequence and 2) pathogen gtf file (not mandatory"
- [readme] README FASTA-header rule: "In the fasta file the header will be used to quantify the number of pathogen total reads, it is recommended that if the fasta header is a complicated name it be simplified"
- [readme] README SARS-CoV-2 reference pointer: "The Sars-CoV-2 reference genome and gtf file can be found [here](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_009858895.2/)"
- [abstract] Article multi-virus capability: "can differentiate and define multiple viruses in a single sample scRNAseq output"
- [readme] testSets co-infection genome folder: [evidence span withheld — anchor status: external_doc]
