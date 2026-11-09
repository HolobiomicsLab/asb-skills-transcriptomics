---
name: single-cell-rna-seq-alignment-and-pcr-duplicate-removal
description: Use when input is full-length scRNA-seq FASTQ data (e.g., SMART-seq2,
  such as the E-MTAB-9801 human fetal bone marrow dataset) and per-cell sorted BAM
  alignments free of PCR duplicates are needed for downstream isoform/IPA analysis.
when_to_use_negative:
- Input is 10x Genomics or other UMI-based droplet scRNA-seq — duplicate marking should
  rely on UMIs (e.g., alevin-fry) rather than samtools rmdup positional duplicate
  removal
- The goal is only gene-level expression quantification from bulk RNA-seq, where duplicate
  removal choices differ and IPA identification is not required
- Reads are already aligned and duplicate-marked BAMs are provided as input — re-aligning
  and re-running rmdup could remove legitimate reads
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3320
tools:
- name: InPACT
  role: Consumes the sorted, deduplicated BAM (with RefSeq GTF and InPACT_polyAsites.hg38.saf)
    to identify skipped and composite IPA sites; default thresholds 5 spliced reads
    (skipped) and 10 spanning reads (composite)
  repo: https://github.com/YY-TMU/InPACT
- name: salmon
  role: Downstream transcript-level quantification of the InPACT_transcript merged
    GTF (k-mer 31 index) to compute per-cell IPA usage after alignment is complete
  repo: https://github.com/COMBINE-lab/salmon
- name: samtools
  role: rmdup removes PCR duplicates; also used to filter out secondary alignments
    per the article's read-filtering steps
- name: HISAT2
  role: Aligns each cell's SMART-seq2 reads to the GRCh38 reference genome
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:0f69e030af75b44d77377f3b5abb6beb870cb6d1474eb31bbc561d6e10e7b42a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - HISAT2
  - InPACT
  - salmon
  - samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# single-cell-rna-seq-alignment-and-pcr-duplicate-removal

## Summary

Preprocessing skill for full-length single-cell RNA-seq (SMART-seq2): align each cell's reads to the GRCh38 reference with HISAT2 and remove PCR duplicates with samtools rmdup, producing coordinate-sorted BAMs suitable for IPA site identification with InPACT.

## When to use

Use when input is full-length scRNA-seq FASTQ data (e.g., SMART-seq2, such as the E-MTAB-9801 human fetal bone marrow dataset) and per-cell sorted BAM alignments free of PCR duplicates are needed for downstream isoform/IPA analysis.

## When NOT to use

- Input is 10x Genomics or other UMI-based droplet scRNA-seq — duplicate marking should rely on UMIs (e.g., alevin-fry) rather than samtools rmdup positional duplicate removal
- The goal is only gene-level expression quantification from bulk RNA-seq, where duplicate removal choices differ and IPA identification is not required
- Reads are already aligned and duplicate-marked BAMs are provided as input — re-aligning and re-running rmdup could remove legitimate reads

## Inputs

- SMART-seq2 single-cell RNA-seq FASTQ files (one sample per cell, e.g., 486 cells from E-MTAB-9801)
- GRCh38 reference genome (FASTA and HISAT2 index)
- Gene annotation GTF (e.g., hg38.109.20211119.ncbiRefSeq.gtf from UCSC goldenPath) for downstream InPACT runs
- Precomputed polyA site annotation SAF (InPACT_polyAsites.hg38.saf)

## Outputs

- Per-cell coordinate-sorted BAM files with PCR duplicates removed
- Pooled BAM across cells ready for IPA site identification with InPACT
- InPACT predict.result.txt listing skipped/composite IPA sites, terminal exons, genes, and polyA sites

## How to apply

Download the SMART-seq2 scRNA-seq dataset (e.g., human fetal bone marrow, 486 cells, from EMBL-EBI ArrayExpress under accession E-MTAB-9801) and align each cell's reads independently to the GRCh38 reference genome with HISAT2. Remove PCR duplicates from each alignment using samtools rmdup, and ensure the BAM is coordinate-sorted as required by InPACT's --input_file/-i option. Optionally remove rRNA-mapped reads with Bowtie and filter secondary alignments with SAMtools, as described in the article's preprocessing of RNA-seq alignments. The resulting per-cell sorted BAMs are then pooled or run through InPACT with the RefSeq GRCh38 GTF and the precomputed InPACT_polyAsites.hg38.saf annotation to identify novel IPA sites. Rationale: duplicate removal prevents amplication artifacts from low-input SMART-seq2 libraries from inflating read support for candidate intronic terminal exons, whose default thresholds are 5 spliced reads (skipped) and 10 spanning reads (composite).

## Related tools

- **InPACT** (Consumes the sorted, deduplicated BAM (with RefSeq GTF and InPACT_polyAsites.hg38.saf) to identify skipped and composite IPA sites; default thresholds 5 spliced reads (skipped) and 10 spanning reads (composite)) — https://github.com/YY-TMU/InPACT
- **salmon** (Downstream transcript-level quantification of the InPACT_transcript merged GTF (k-mer 31 index) to compute per-cell IPA usage after alignment is complete) — https://github.com/COMBINE-lab/salmon
- **samtools** (rmdup removes PCR duplicates; also used to filter out secondary alignments per the article's read-filtering steps)
- **HISAT2** (Aligns each cell's SMART-seq2 reads to the GRCh38 reference genome)

## Examples

```
hisat2 -x GRCh38 -U cell_1.fq | samtools sort -o cell_1.sorted.bam && samtools rmdup cell_1.sorted.bam cell_1.dedup.bam
```

## Evaluation signals

- Each output BAM is coordinate-sorted and passes 'samtools quickcheck'; duplicate counts reported by samtools rmdup are consistent with the low-input SMART-seq2 library expectations
- Running InPACT on the pooled BAM with InPACT_polyAsites.hg38.saf and default thresholds (5 spliced / 10 spanning reads) produces a predict.result.txt containing skipped and composite terminal exons
- Applied to the full 486-cell E-MTAB-9801 dataset, the downstream pipeline should recover on the order of 2635 novel IPA events in 2157 genes (599 skipped, 2076 composite), with 533 cell-type-specific events after TPM < 1 filtering
- Output tables are valid TSV files containing IPA events, cell-type-specific assignments, and GO enrichment terms matching the reported counts

## Limitations

- No version number given for InPACT or any compared tool in the article, so exact reproducibility of tool versions is uncertain
- The article discussion does not state the dataset accession; E-MTAB-9801 must be obtained from the task metadata rather than the paper text
- samtools rmdup positional duplicate removal is appropriate for full-length SMART-seq2 data but would incorrectly collapse UMI-based droplet data duplicates

## Evidence

- [other] align each cell's reads to the GRCh38 reference with HISAT2, removing PCR duplicates with samtools rmdup: [evidence span withheld — anchor status: not_found]
- [methods] rmdup evidence: "followed by the removal of PCR duplicates using samtools rmdup"
- [methods] secondary alignment filter: "Reads flagged as secondary alignment were filtered out using SAMtools"
- [readme] InPACT CLI from README: "InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5"
- [readme] sorted BAM requirement from README: "A BAM file that would be used to predict IPA sites and it should be sorted by coordinates."
