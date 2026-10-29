---
name: smart-seq2-alignment-deduplication
description: Use when starting from raw Smart-seq2 scRNA-seq FASTQ files that must
  become analysis-ready BAMs before variant pileup (e.g. for the MQuad cellSNP–MQuad–vireoSNP
  clonality pipeline).
when_to_use_negative:
- Input is already an analysis-ready BAM (e.g. 10x BAMs directly downloaded, as for
  the MKN-45 scDNA-seq and TNBC datasets) — skip to cellsnp-lite pileup
- Data are mtscATAC-seq reads processed with CellRanger-ATAC (e.g. the CRC dataset),
  where NUMT-masked reference handling rather than STAR alignment is needed
- Libraries carry UMIs (e.g. 10x scRNA-seq), where duplicate handling is done via
  UMI/barcode flags rather than Picard MarkDuplicates on alignments
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: STAR
  role: Aligns Smart-seq2 FASTQ files to the UCSC hg19 reference genome with default
    parameters
- name: Picard MarkDuplicates
  role: Marks and removes PCR duplicates from raw STAR-aligned BAMs (v2.18.9) to produce
    analysis-ready BAMs
- name: cellsnp-lite
  role: 'Downstream consumer: piles up mitochondrial reads from the deduplicated BAMs
    into SNP-by-cell AD/DP matrices (v1.2.1)'
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: cellSNP
  role: Alternative pileup tool; its README notes that PCR duplicate filtering flags
    are unreliable for CellRanger-style output, motivating careful duplicate handling
    upstream
  repo: https://github.com/single-cell-genetics/cellSNP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:c0774d764ba511ef63ef6351f090d5d434aa74a6c33d827b9e1129448aecb3c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/single-cell-genetics/cellsnp-lite
  tools:
  - Cardelino
  - MQuad
  - Picard
  - Picard MarkDuplicates
  - STAR
  - cellSNP
  - cellsnp-lite
  - kneed
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# smart-seq2-alignment-deduplication

## Summary

Preprocessing skill for full-length Smart-seq2 scRNA-seq FASTQs: align reads to the UCSC hg19 reference with STAR (default parameters) and mark/remove PCR duplicates from the raw BAMs with Picard MarkDuplicates v2.18.9, producing analysis-ready BAMs for downstream mtDNA pileup with cellsnp-lite.

## When to use

Use when starting from raw Smart-seq2 scRNA-seq FASTQ files that must become analysis-ready BAMs before variant pileup (e.g. for the MQuad cellSNP–MQuad–vireoSNP clonality pipeline).

## When NOT to use

- Input is already an analysis-ready BAM (e.g. 10x BAMs directly downloaded, as for the MKN-45 scDNA-seq and TNBC datasets) — skip to cellsnp-lite pileup
- Data are mtscATAC-seq reads processed with CellRanger-ATAC (e.g. the CRC dataset), where NUMT-masked reference handling rather than STAR alignment is needed
- Libraries carry UMIs (e.g. 10x scRNA-seq), where duplicate handling is done via UMI/barcode flags rather than Picard MarkDuplicates on alignments

## Inputs

- Per-cell FASTQ files from a Smart-seq2 scRNA-seq experiment (e.g. the joxm healthy fibroblast dataset)
- UCSC hg19 reference genome (or mm10) and STAR genome index
- Picard v2.18.9 MarkDuplicates tool

## Outputs

- Per-cell coordinate-sorted alignment BAM files (analysis-ready)
- Picard MarkDuplicates metrics files
- BAM list with sample/cell IDs ready for cellsnp-lite pileup of mitochondrial reads

## How to apply

First build (or reuse) a STAR genome index for the UCSC hg19 reference genome (mm10 for mouse) and align each cell's FASTQ files with STAR using default parameters, emitting coordinate-sorted BAM files. Then run Picard MarkDuplicates (v2.18.9) on each raw BAM to mark and remove PCR duplicates, since Smart-seq2 libraries have no UMIs so duplicate reads cannot be collapsed at the UMI level. Verify BAM integrity (header, cell/sample ID, sorted order) before handing the analysis-ready BAMs to cellsnp-lite for mitochondrial pileup into SNP-by-cell AD/DP matrices. This alignment-plus-dedup step is the prerequisite for MQuad's binomial mixture model to distinguish clonally informative mtDNA variants from sequencing noise, which relies on accurate per-cell depths.

## Related tools

- **STAR** (Aligns Smart-seq2 FASTQ files to the UCSC hg19 reference genome with default parameters)
- **Picard MarkDuplicates** (Marks and removes PCR duplicates from raw STAR-aligned BAMs (v2.18.9) to produce analysis-ready BAMs)
- **cellsnp-lite** (Downstream consumer: piles up mitochondrial reads from the deduplicated BAMs into SNP-by-cell AD/DP matrices (v1.2.1)) — https://github.com/single-cell-genetics/cellsnp-lite
- **cellSNP** (Alternative pileup tool; its README notes that PCR duplicate filtering flags are unreliable for CellRanger-style output, motivating careful duplicate handling upstream) — https://github.com/single-cell-genetics/cellSNP

## Examples

```
STAR --genomeDir hg19_star_index --readFilesIn cell_R1.fastq cell_R2.fastq --readFilesCommand zcat --outSAMtype BAM SortedByCoordinate --outFileNamePrefix cell1_ ; picard MarkDuplicates I=cell1_Aligned.sortedByCoord.out.bam O=cell1_dedup.bam M=cell1_dup_metrics.txt REMOVE_DUPLICATES=true
```

## Evaluation signals

- Each cell yields a valid, coordinate-sorted BAM with correct RG/cell-ID tags after STAR alignment
- Picard MarkDuplicates metrics report a nonzero duplicate fraction consistent with Smart-seq2 (no-UMI) libraries, and duplicate reads are excluded from the final BAMs
- The analysis-ready BAMs support mitochondrial pileup with cellsnp-lite producing AD/DP matrices with usable depth for downstream MQuad ΔBIC model selection
- Reference genome build matches downstream expectations (hg19 for the fibroblast/Kim datasets; hg38-only references such as CellRanger outputs are explicitly incompatible)

## Limitations

- Smart-seq2 data lack UMIs, so PCR duplicates can only be detected by positional MarkDuplicates; the cellSNP README warns that duplicate-flag filtering 'may lead to unexpected coverage reduction in some cases', so duplicate handling choices propagate to variant coverage
- Reference build matters: MQuad's published processed data use UCSC hg19/mm10, while some public BAMs are aligned to hg38 and differ
- This step only produces clean BAMs — it does not call variants; clonally informative mtDNA SNV selection requires the subsequent cellsnp-lite → MQuad → vireoSNP pipeline

## Evidence

- [discussion] STAR alignment with default parameters to UCSC hg19: "FASTQ files were first aligned to the reference genome downloaded from UCSC (hg19 for human, mm10 for mouse) using STAR aligner"
- [discussion] Picard MarkDuplicates duplicate removal: "duplicates were marked and removed from the raw BAM files using MarkDuplicates from Picard (v2.18.9)"
- [discussion] Downstream pileup consumes the deduplicated BAMs: "raw reads from BAM files are piled up using cellSNP-lite (v1.2.1)23. This generates a SNP-by-cell matrix"
- [readme] Duplicate-flag caveats for pileup tools: [evidence span withheld — anchor status: external_doc]
