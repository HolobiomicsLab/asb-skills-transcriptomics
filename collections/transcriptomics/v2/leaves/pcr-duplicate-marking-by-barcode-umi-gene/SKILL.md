---
name: pcr-duplicate-marking-by-barcode-umi-gene
description: Use when converting aligned 10x droplet scRNA-seq reads (Read1/cDNA insert
  aligned with STAR, MAPQ=255) into a per-cell UMI count matrix and PCR duplicates
  must be removed before counting.
when_to_use_negative:
- Input is already a gene-cell UMI count matrix or feature table — duplicate marking
  has already been applied upstream by Cell Ranger
- Sequencing data are not from a 10x/GemCode droplet 3' assay with cell barcode +
  UMI read structure (e.g. full-length plate-based scRNA-seq without UMIs)
- Downstream analysis such as SNV calling or chimerism inference where read-level
  rather than molecule-level information is required
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing and single-cell 3' gene
    counting, including PCR-duplicate marking by (cell barcode, UMI, gene ID) and
    gene-barcode matrix generation
  repo: https://github.com/10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 (the cDNA insert) to hg19, producing the confidently mapped (MAPQ=255)
    alignments on which duplicate marking is performed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:2a8670d44c128f6b49f8dbc6d3b3d19c013b3bdf46e9fd436a02f13b407b83f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - SRA SRP073767
  - STAR
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# pcr-duplicate-marking-by-barcode-umi-gene

## Summary

Marks and removes PCR duplicates from 10x Genomics single-cell 3' reads by collapsing reads sharing the same (cell barcode, UMI, gene ID) tuple, so that only confidently mapped, non-duplicate reads with valid barcodes and UMIs contribute to the gene-barcode count matrix.

## When to use

Use when converting aligned 10x droplet scRNA-seq reads (Read1/cDNA insert aligned with STAR, MAPQ=255) into a per-cell UMI count matrix and PCR duplicates must be removed before counting.

## When NOT to use

- Input is already a gene-cell UMI count matrix or feature table — duplicate marking has already been applied upstream by Cell Ranger
- Sequencing data are not from a 10x/GemCode droplet 3' assay with cell barcode + UMI read structure (e.g. full-length plate-based scRNA-seq without UMIs)
- Downstream analysis such as SNV calling or chimerism inference where read-level rather than molecule-level information is required

## Inputs

- Raw FASTQ files from a deposited sample (SRA SRP073767)
- hg19 human reference genome
- STAR alignments of Read1 (cDNA insert) restricted to MAPQ=255
- Cell barcodes with per-candidate posterior probabilities and UMIs with base quality scores

## Outputs

- Gene-by-cell-barcode UMI count matrix (e.g. gene_barcode_matrix.csv) built only from confidently mapped, non-PCR-duplicate reads with valid barcodes and UMIs

## How to apply

After aligning Read1 (the cDNA insert) to hg19 with STAR and retaining only confidently mapped alignments (MAPQ=255), validate each read's cell barcode: correct it when the posterior probability of a candidate barcode is at least 0.975 (choosing the highest-posterior candidate), otherwise discard the read. Check UMI validity, keeping UMIs with sequencing quality score >10 that are not homopolymers, and collapse UMIs 1-Hamming-distance away from a more abundant UMI for the same cell barcode and gene. Then mark PCR duplicates as reads sharing the same (cell barcode, UMI, gene ID) tuple and retain only one read per tuple. Tabulate the surviving non-duplicate reads into a gene-by-cell-barcode UMI count matrix; the rationale is that each (barcode, UMI, gene) tuple represents one original mRNA molecule, so duplicate reads are amplification artifacts that would otherwise inflate counts.

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing and single-cell 3' gene counting, including PCR-duplicate marking by (cell barcode, UMI, gene ID) and gene-barcode matrix generation) — https://github.com/10XGenomics/cellranger
- **STAR** (Aligns Read1 (the cDNA insert) to hg19, producing the confidently mapped (MAPQ=255) alignments on which duplicate marking is performed)

## Examples

```
cellranger count --id=sample1 --transcriptome=hg19 --fastqs=SRP073767_fastqs --expect-cells=8000
```

## Evaluation signals

- Every count in the output matrix derives from a read with MAPQ=255, a valid barcode, and a valid UMI (quality >10, non-homopolymer)
- No two retained reads share the same (cell barcode, UMI, gene ID) tuple — one read per tuple after deduplication
- Barcodes in the output were either corrected at posterior probability >=0.975 or originally valid; reads with uncorrectable barcodes were discarded
- UMI counts per cell barcode and gene reflect collapsed 1-Hamming-distance UMIs corrected to the more abundant UMI
- Matrix dimensions match the gene set of the hg19 reference and the set of accepted cell barcodes (top barcodes >10% of the top nth barcode, n = 1% of expected recovered cell count)

## Limitations

- Reads whose cell barcode cannot be corrected with posterior probability >=0.975 are discarded, losing data from low-quality barcoded molecules
- UMI correction by 1-Hamming-distance collapsing can merge genuinely distinct UMIs when UMI space is saturated
- PCR-duplicate marking counts all reads in a (barcode, UMI, gene) tuple as one molecule, so UMI collisions (distinct molecules sharing a UMI) cannot be resolved by this rule alone
- Some earlier libraries used a 5 nt UMI with 5 bp Read2 instead of the standard structure, introducing protocol heterogeneity across libraries

## Evidence

- [discussion] Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix.: "Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix."
- [discussion] If the posterior probability for any candidate barcode is at least 0.975, then the barcode is corrected to the candidate barcode with the highest posterior probability.: "If the posterior probability for any candidate barcode is at least 0.975, then the barcode is corrected to the candidate barcode with the highest posterior probability."
- [discussion] UMIs with sequencing quality score >10 were considered valid if they were not homopolymers.: "UMIs with sequencing quality score >10 were considered valid if they were not homopolymers."
- [discussion] A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads.: "A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads."
- [readme] The Cell Ranger Single-Cell Software Suite was used to perform sample demultiplexing, barcode processing and single-cell 3′ gene counting: "The Cell Ranger Single-Cell Software Suite was used to perform sample demultiplexing, barcode processing and single-cell 3′ gene counting"
