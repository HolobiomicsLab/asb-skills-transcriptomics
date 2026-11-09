---
name: transcript-assembly-benchmark-evaluation-against-refseq-annotation
description: Use when you have GTF transcript assemblies from multiple assemblers
  run on the same RNA-seq alignments (e.g., Hisat2/STAR BAMs) and need to quantify
  which assembler recovers more reference-annotated (RefSeq) transcripts correctly.
when_to_use_negative:
- Input is long-read RNA-seq data (PacBio or ONT) — Tiglon's current version is not
  compatible with long-read datasets
- You want annotation-guided assembly — the current Tiglon version has no option to
  use known transcript annotations to guide assembly
- The goal is de novo transcript discovery without a reference genome or RefSeq annotation
  to compare against
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Tiglon
  role: Integrates Hisat2 and STAR alignments via labeled splice graphs to assemble
    candidate transcripts evaluated in the benchmark
  repo: https://github.com/yutingsdu/Tiglon
- name: StringTie2
  role: Baseline assembler run per alignment condition and with --merge to build the
    merged baseline assembly
  repo: https://github.com/gpertea/stringtie
- name: Scallop
  role: Baseline assembler compared for correct-transcript counts on Hisat2/STAR/merged
    alignments
  repo: https://github.com/Kingsford-Group/scallop
- name: Hisat2
  role: Produces splice-aware BAM alignments with --dta for one alignment condition
  repo: https://github.com/DaehwanKimLab/hisat2
- name: STAR
  role: Produces the second splice-aware BAM alignment condition
  repo: https://github.com/alexdobin/STAR
- name: bamtools
  role: Prerequisite library required to build and run Tiglon on BAM inputs
  repo: https://github.com/pezmaster31/bamtools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.104067
    title: Tiglon enables accurate transcriptome assembly via integrating mappings
      of different aligners
  source_content_hash: sha256:30e6100223b19f2a1504cea8d31ae3cc2627cbb13ee5fae8d5bf18dcd772bc9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yutingsdu/Tiglon
  tools:
  - Hisat2
  - STAR
  - Scallop
  - Star
  - StringTie2
  - Tiglon
  - bamtools
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# transcript-assembly-benchmark-evaluation-against-refseq-annotation

## Summary

Benchmark genome-guided transcriptome assemblers (Tiglon, StringTie2, Scallop, StringTie2-Merge, iPAC, TransBorrow) by evaluating their GTF assemblies with Cuffcompare against hg38.ncbiRefSeq.gtf, counting correct transcripts and precision per assembler. Enables reproduction of the Tiglon paper's reported 9.51%/11.25%–13.68% advantages on eight H. sapiens datasets.

## When to use

Use when you have GTF transcript assemblies from multiple assemblers run on the same RNA-seq alignments (e.g., Hisat2/STAR BAMs) and need to quantify which assembler recovers more reference-annotated (RefSeq) transcripts correctly.

## When NOT to use

- Input is long-read RNA-seq data (PacBio or ONT) — Tiglon's current version is not compatible with long-read datasets
- You want annotation-guided assembly — the current Tiglon version has no option to use known transcript annotations to guide assembly
- The goal is de novo transcript discovery without a reference genome or RefSeq annotation to compare against

## Inputs

- Eight H. sapiens RNA-seq FASTQ files (SRA: SRR307911, SRR387662, SRR10517380, ERR2403203, SRR307903, SRR315323, SRR315334, SRR7536920)
- Splice-aware BAM alignments from Hisat2 (--dta) and STAR on GRCh38, plus samtools-merged combined BAMs and a bamFiles_list (one BAM per line)
- Assembled GTF files from Tiglon, StringTie2, Scallop, StringTie2-Merge, iPAC, and TransBorrow
- Reference annotation hg38.ncbiRefSeq.gtf (NCBI RefSeq on GRCh38)

## Outputs

- Cuffcompare evaluation of each assembler GTF against hg38.ncbiRefSeq.gtf (correct transcript counts, precision)
- Aggregated benchmark table human_benchmark.csv with per-assembler correct transcripts, precision, and low-expression-transcript recovery
- value_in_range validation checks confirming Tiglon's reported 9.51% and 11.25%–13.68% advantages

## How to apply

Align each RNA-seq FASTQ (e.g., SRA accessions SRR307911, SRR387662, SRR10517380, ERR2403203, SRR307903, SRR315323, SRR315334, SRR7536920) to GRCh38 with Hisat2 (--dta) and STAR, and also merge aligner-specific BAMs with samtools to form a combined-alignment condition. Run each assembler under matching Hisat2/STAR/merged conditions — Tiglon with default filters (min_trans_length 500, min_average_frac 0.03, min_unbalance_frac 0.03, min_gap_length 200), plus StringTie2, Scallop, iPAC, TransBorrow, and StringTie2 --merge. Evaluate all resulting GTFs with Cuffcompare against hg38.ncbiRefSeq.gtf, counting correct transcripts and precision per assembler per dataset. Aggregate the per-assembler correct-transcript counts, precision, and low-expression-transcript recovery across the eight datasets into human_benchmark.csv. Validate by checking the reproduced advantages fall in the reported ranges: ~9.51% more correct transcripts than StringTie2-Merge and 11.25%–13.68% more than StringTie2 and Scallop on merged alignments.

## Related tools

- **Tiglon** (Integrates Hisat2 and STAR alignments via labeled splice graphs to assemble candidate transcripts evaluated in the benchmark) — https://github.com/yutingsdu/Tiglon
- **StringTie2** (Baseline assembler run per alignment condition and with --merge to build the merged baseline assembly) — https://github.com/gpertea/stringtie
- **Scallop** (Baseline assembler compared for correct-transcript counts on Hisat2/STAR/merged alignments) — https://github.com/Kingsford-Group/scallop
- **Hisat2** (Produces splice-aware BAM alignments with --dta for one alignment condition) — https://github.com/DaehwanKimLab/hisat2
- **STAR** (Produces the second splice-aware BAM alignment condition) — https://github.com/alexdobin/STAR
- **bamtools** (Prerequisite library required to build and run Tiglon on BAM inputs) — https://github.com/pezmaster31/bamtools

## Examples

```
Tiglon -B bamFiles_list -s first -o Tiglon_outdir -p 2
```

## Evaluation signals

- human_benchmark.csv shows Tiglon with ~9.51% more correct transcripts than StringTie2-Merge averaged over the eight datasets (value_in_range check)
- Tiglon's advantage over StringTie2 and Scallop on merged alignments falls in the reported 11.25%–13.68% range
- Each assembler's GTF passes Cuffcompare comparison against hg38.ncbiRefSeq.gtf producing nonzero correct-transcript counts and precision values
- Tiglon's default filters were applied as specified: min_trans_length 500, min_average_frac 0.03, min_unbalance_frac 0.03, min_gap_length 200
- bamFiles_list contains one BAM per line covering both Hisat2 and STAR alignment sets for the merged condition

## Limitations

- The current Tiglon version does not support annotation-guided assemblies where known transcript annotations guide assembly
- Tiglon is not compatible with long-read RNA-seq datasets (e.g., Pacific Biosciences or Oxford Nanopore Technologies)
- Combining Tiglon's integrating strategy with TransBorrow's borrowing strategy is proposed future work, not implemented
- Tiglon currently only accepts BAM files generated by Hisat(2) and Star

## Evidence

- [intro] Tiglon averagely reconstructed 9.51% more correct transcripts than StringTie2-Merge, 11.25%–13.68% more than StringTie2 and Scallop: "Tiglon averagely reconstructed 9.51% more correct transcripts than StringTie2-Merge, 11.25%–13.68% more than StringTie2 and Scallop"
- [abstract] Tiglon integrates multiple alignments and builds labeled splice graphs: "integrates multiple alignments of different mapping tools and builds the labeled splice graphs"
- [methods] Tiglon filter defaults: "--min_trans_length/-L <int>   	: Minimum assembled transcript length, default: 500."
- [discussion] No annotation-guided assembly in current Tiglon: "The current version of Tiglon does not have an option for annotation-guided assemblies, i.e., where the known transcript annotations are used to guide the assembly procedure."
- [readme] Tiglon input format constraint: "The current version of Tiglon can accepts BAM files generated by Hisat(2) and Star."
