---
name: circrna-call-coordinate-intersection
description: Use when you have circRNA call sets from ≥2 detection pipelines on the
  same ribo− RNA-seq data and need coordinate-based overlaps, Venn counts, or per-circRNA
  BSJ fragment comparisons.
when_to_use_negative:
- Call sets come from different genome builds or annotations, so coordinate intersection
  would be spurious
- Only one circRNA detection tool was run and no cross-tool comparison is possible
- The goal is quantifying circ vs linear expression (FPB/CIRCscore) rather than comparing
  detection tools — use the CLEAR quantification workflow instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: CIRCexplorer3-CLEAR
  role: Produces the CIRCexplorer2-based circRNA call set with readNumber and FPB
    fields via clear_quant
  repo: https://github.com/YangLab/CLEAR
- name: CIRCexplorer2
  role: Calls circRNAs (version >=2.3.6) within the CLEAR pipeline for one arm of
    the comparison
  repo: https://github.com/YangLab/CIRCexplorer2
- name: HISAT2
  role: Aligns the paired ribo− RNA-seq reads (>=2.0.5); its BAM is shared input to
    DCC and circ_quant
  repo: https://ccb.jhu.edu/software/hisat2/index.shtml
- name: DCC/circtools
  role: Independent circRNA detector whose calls are intersected with the CIRCexplorer2/MapSplice
    sets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2019.11.004
    title: 'CIRCexplorer3: A CLEAR Pipeline for Direct Comparison of Circular and
      Linear RNA Expression'
  source_content_hash: sha256:655120db22badad45116320d1b9f35e01fb3c839932c7a3ad0c4f03aff8a08d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLab/CLEAR
  tools:
  - CIRCexplorer2
  - CIRCexplorer3-CLEAR
  - DCC/circtools
  - HISAT2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# circRNA-call-coordinate-intersection

## Summary

Intersects circRNA call sets from multiple pipelines (CIRCexplorer2/CLEAR, MapSplice, DCC/circtools) by genomic coordinates (chrom, start, end, strand) to compute pairwise and three-way overlap counts and compare per-tool BSJ fragment support.

## When to use

Use when you have circRNA call sets from ≥2 detection pipelines on the same ribo− RNA-seq data and need coordinate-based overlaps, Venn counts, or per-circRNA BSJ fragment comparisons.

## When NOT to use

- Call sets come from different genome builds or annotations, so coordinate intersection would be spurious
- Only one circRNA detection tool was run and no cross-tool comparison is possible
- The goal is quantifying circ vs linear expression (FPB/CIRCscore) rather than comparing detection tools — use the CLEAR quantification workflow instead

## Inputs

- Paired-end ribo− RNA-seq FASTQ files (PA1 samples)
- hg38 genome FASTA, annotation GTF, HISAT2 and bowtie1 indexes
- CIRCexplorer2/CLEAR circRNA call table (quant.txt with readNumber field)
- MapSplice circRNA call set
- DCC/circtools circRNA calls from the HISAT2 BAM

## Outputs

- circrna_tool_overlap_counts.tsv with Venn-style pairwise and three-way overlap counts (356 CIRCexplorer2-and-MapSplice-only; 787 three-way)
- Per-circRNA per-tool BSJ fragment (readNumber) counts for the overlapping circRNAs

## How to apply

Generate each tool's call set on the same PA1 ribo− RNA-seq FASTQ inputs: CLEAR's clear_quant with CIRCexplorer2 (>=2.3.6) after HISAT2 (>=2.0.5) alignment, MapSplice on the same FASTQ files, and circtools-embedded DCC on the HISAT2 BAM (which must not contain unmapped reads). Apply a uniform filter of >= 3 BSJ-supporting fragments (readNumber >= 3) to every call set so comparisons are on equal footing. Intersect the filtered sets by genomic coordinates (chrom, start, end, strand) to compute pairwise and three-way overlap counts, then tabulate per-circRNA per-tool readNumber values to reveal differences in BSJ fragment calling efficiency. In the reported benchmark this recovers 356 circRNAs detected by both CIRCexplorer2 and MapSplice but missed by DCC, and 787 three-way overlapping circRNAs among which DCC generally called fewer BSJ-mapped fragments. Write the Venn-style counts and per-circRNA per-tool fragment counts to circrna_tool_overlap_counts.tsv.

## Related tools

- **CIRCexplorer3-CLEAR** (Produces the CIRCexplorer2-based circRNA call set with readNumber and FPB fields via clear_quant) — https://github.com/YangLab/CLEAR
- **CIRCexplorer2** (Calls circRNAs (version >=2.3.6) within the CLEAR pipeline for one arm of the comparison) — https://github.com/YangLab/CIRCexplorer2
- **HISAT2** (Aligns the paired ribo− RNA-seq reads (>=2.0.5); its BAM is shared input to DCC and circ_quant) — https://ccb.jhu.edu/software/hisat2/index.shtml
- **DCC/circtools** (Independent circRNA detector whose calls are intersected with the CIRCexplorer2/MapSplice sets)

## Examples

```
clear_quant -1 mate_1.fastq -2 mate_2.fastq -g hg38.fa -i hg38.hisat_index -j hg38.bowtie_index -G annotation.gtf -o output_dir
```

## Evaluation signals

- The overlap table contains the count 356 for the CIRCexplorer2-and-MapSplice-only intersection (matches Figure 3B)
- The overlap table contains the count 787 for the three-way overlap (matches Figure 3B)
- All three call sets were filtered to >= 3 BSJ-supporting fragments before intersection
- Every intersected circRNA matches on chrom, start, end, and strand across tools
- Per-circRNA per-tool readNumber tabulation shows DCC calling fewer BSJ-mapped fragments among the 787 overlapping circRNAs (Figure 3C)

## Limitations

- DCC's BAM input must not contain unmapped reads
- The >= 3 BSJ fragment cutoff restricts the comparison to relatively supported circRNAs and may exclude low-support calls
- CIRCscore is not applicable to poly(A)−/ribo− or RNase R-treated datasets, limiting cross-dataset extension of the comparison
- Coordinate intersection is coordinate-stringency dependent; call sets with differing boundary definitions can undercount overlaps

## Evidence

- [results] 356 circRNAs (BSJ fragments ≥ 3) identified by both CIRCexplorer2 and MapSplice were missed by DCC: [evidence span withheld — anchor status: not_found]
- [readme] clear_quant usage: "usage: clear_quant [-h] -1 M1 [-2 M2] -g GENOME -i HISAT -j BOWTIE1 -G GTF"
- [readme] BAM must not contain unmapped reads: "*hisat_aligned.bam should not contain unmapped reads.*"
- [methods] Install CLEAR: "git clone https://github.com/YangLab/CLEAR
cd CLEAR
python ./setup.py install"
- [readme] Output fields including readNumber: "| readNumber  | Number of junction reads              |"
