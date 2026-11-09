---
name: serum-small-rna-expression-ranking
description: Use when raw serum/plasma small RNA-seq FASTQ files need adapter trimming,
  quality filtering, genome alignment, small RNA annotation, and a ranked RNA-by-sample
  read count matrix, especially when miRNAs and other small RNA classes (<200 nt)
  must be profiled together.
when_to_use_negative:
- The goal is microbial profiling of unmapped reads — that requires the separate Microbe
  module (BLAST against UCSC nt) and additional blast_* databases
- The input is already a count matrix or expression table — the QC/Alignment/Annotation
  modules would redundantly reprocess raw reads
- Case/control differential expression testing is the sole goal without raw FASTQ
  inputs — the Function module's Mann-Whitney (mwu) test operates on the aggregated
  count matrix and cannot be applied without upstream COMPSRA counts
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: COMPSRA
  role: End-to-end five-module pipeline (QC, Alignment, Annotation, Microbe, Function)
    that processes serum FASTQ files into small RNA count matrices
  repo: https://github.com/cougarlj/COMPSRA
- name: STAR
  role: Default aligner used by the COMPSRA Alignment module to map qualified serum
    reads to hg38/hg19
- name: miRBase
  role: Reference database used by the Annotation module to identify miRNAs among
    human-genome-mapped reads
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:eae057ebcc62e7554b7cf4df32415d8ebfd5f4b633b9b19d63e7fbdbaef35b25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - STAR
  - miRBase
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# serum-small-rna-expression-ranking

## Summary

Identifies and ranks small RNA molecules (miRNA, piRNA, snRNA, snoRNA, tRNA, circRNA) by abundance from serum small RNA-seq FASTQ files using the COMPSRA five-module pipeline (QC, Alignment, Annotation, Function), producing per-sample RNA count matrices. Used to characterize the diversity and abundance of circulating small RNAs in low-input serum samples.

## When to use

Use when raw serum/plasma small RNA-seq FASTQ files need adapter trimming, quality filtering, genome alignment, small RNA annotation, and a ranked RNA-by-sample read count matrix, especially when miRNAs and other small RNA classes (<200 nt) must be profiled together.

## When NOT to use

- The goal is microbial profiling of unmapped reads — that requires the separate Microbe module (BLAST against UCSC nt) and additional blast_* databases
- The input is already a count matrix or expression table — the QC/Alignment/Annotation modules would redundantly reprocess raw reads
- Case/control differential expression testing is the sole goal without raw FASTQ inputs — the Function module's Mann-Whitney (mwu) test operates on the aggregated count matrix and cannot be applied without upstream COMPSRA counts

## Inputs

- Serum/plasma small RNA-seq FASTQ files (e.g. 12 healthy-control serum samples, 50 nt reads)
- 3' adapter sequence (TGGAATTCTCGGGTGCCAAGG) and ligation random-base count
- Prebuilt annotation databases: miRNA_hg38, piRNA_hg38, tRNA_hg38, snoRNA_hg38, snRNA_hg38, circRNA_hg38
- hg38 STAR reference genome index

## Outputs

- Per-sample tab-delimited read count files per RNA type from the Annotation module
- Aggregated RNA-molecule-by-sample count matrix (Function module, -fm/-fms)
- CpM-normalized expression matrix (optional, -fdnorm cpm)
- Qualified/clean reads (FASTQ) and STAR-aligned BAM (Aligned.out.bam) per sample

## How to apply

Run the COMPSRA Quality Control module on each serum FASTQ with the 3' adapter TGGAATTCTCGGGTGCCAAGG, removal of 4 random ligation bases (-rb 4), and quality thresholds (-rh 20 -rt 20 -rr 20), then restrict reads to the small-RNA size window via length grouping (-rlh 8,17). Align qualified reads to hg38 with the Alignment module using STAR (local read-ends alignment, --outFilterMismatchNmax 1), building the STAR index on first use (-mbi, ~3 hours). Annotate mapped reads with the Annotation module against prebuilt databases (miRBase for miRNA, piRNABank/piRBase/piRNACluster for piRNA, gtRNAdb for tRNA, GENCODE release 27 for snRNA/snoRNA, circBase for circRNA) with a full-overlap criterion (-ac 1, Rolp = 1). Aggregate per-sample counts into an RNA-by-sample matrix with the Function module (-fun -fm -fms), optionally normalizing with Count-per-Million (CpM); the paper's serum benchmark detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs at a read count threshold of 5 across 12 healthy-control serum samples.

## Related tools

- **COMPSRA** (End-to-end five-module pipeline (QC, Alignment, Annotation, Microbe, Function) that processes serum FASTQ files into small RNA count matrices) — https://github.com/cougarlj/COMPSRA
- **STAR** (Default aligner used by the COMPSRA Alignment module to map qualified serum reads to hg38/hg19)
- **miRBase** (Reference database used by the Annotation module to identify miRNAs among human-genome-mapped reads)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -aln -mt star -ann -ac 1 -fun -fm -fms 1-12 -fdclass 1 -fdann -inf ./sample.list -out ./serum_out/
```

## Evaluation signals

- QC output: after trimming the 50 nt raw reads for the adapter plus 4 random bases at both 5' and 3' ends, remaining read lengths should fall in the 0–42 nt range, with retained reads in the 8–17 nt binning window (-rlh 8,17)
- Alignment sanity: STAR Aligned.out.bam files exist for each sample and unmapped reads are written to a FASTA/BAM for downstream modules
- Count matrix shape: RNA-by-sample matrix with one column per FASTQ input (e.g. -fms 1-12 for 12 serum samples), rows partitioned by RNA class
- Abundance benchmark: on the paper's 12 healthy serum samples at read count threshold 5, expect roughly 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs detected
- Normalization check: CpM-normalized values should sum to 1,000,000 per sample column

## Limitations

- First-time STAR index building (-mbi) costs about 3 hours; per-sample processing takes ~10 minutes (vs ~20 minutes for exceRpt) and the paper's run used a server with 30 GB RAM
- Annotation requires prebuilt databases matched to the reference genome version (hg19 vs hg38), reconciled via automatic UCSC LiftOver; mismatched genome/database versions will fail or mis-annotate
- The paper's miRNA results section cites miRBase20 in the article methods while the validation task uses miRBase v21 — background version choices affect detected miRNA counts
- Random ligation bases at both 5' and 3' ends reduce usable read length to 0–42 nt from 50 nt raw reads; very short inserts may be entirely trimmed away

## Evidence

- [results] COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs: "COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs, as well as 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses"
- [methods] QC module adapter and random base trimming: "First, the adapter portions of a read are trimmed along with any randomized bases at ligation junctions that are produced by some small RNA-seq kits"
- [results] The length of raw reads was 50 nt and after trimming adapters and 4 random bases: "The length of raw reads was 50 nt and after trimming adapters and 4 random bases at both 5′ and 3′ ends, the read length varied from 0 nt to 42 nt"
- [readme] COMPSRA pipeline invocation with adapter TGGAATTCTCGGGTGCCAAGG: "java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -aln -mt star -ann -ac 1,2,3,4,5,6 -fun -fm -fms 1-12"
- [methods] miRBase and other annotation databases: "COMPSRA currently uses several different (and expandable) small RNA reference databases for annotating human genome mapped reads: miRBase20 for miRNA"
