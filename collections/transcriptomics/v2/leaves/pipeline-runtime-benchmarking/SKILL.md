---
name: pipeline-runtime-benchmarking
description: Use when two pipelines claim performance differences on the same data
  type and you need a fair head-to-head runtime comparison on a fixed FASTQ subset
  with identical references and comparable settings.
when_to_use_negative:
- You are evaluating analysis quality (clustering accuracy, spatial resolution) rather
  than computational runtime — use output-comparison skills instead.
- The pipelines process different input subsets or reference genomes, making wall-clock
  times non-comparable.
- Input is already a processed DGE/feature table; the benchmark requires raw FASTQ
  processing end-to-end.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: STtools
  role: Pipeline under benchmark; run steps A1-A3 (seqtk coordinate extraction + STARsolo
    alignment) on the SRR14082757 subset
  repo: https://github.com/seqscope/STtools
- name: STARsolo
  role: Alignment engine invoked by STtools Step A3 to produce the DGE; requires STAR
    >=2.7.5c
  repo: https://github.com/alexdobin/STAR
- name: seqtk
  role: Used in Step A1 to process the 1st-seq FASTQ and extract spatial coordinates
    and the STARsolo whitelist
  repo: https://github.com/lh3/seqtk
- name: spacemake
  role: Comparison pipeline run on the identical FASTQ subset with the mm10 reference
    under comparable settings
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:f97606d2c864590242fe3ae2684e8f263f6861102c298f7dafaebb114209c98a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - STARsolo
  - STtools
  - seqtk
  - spacemake
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# pipeline-runtime-benchmarking

## Summary

Benchmark the wall-clock runtime of competing bioinformatics pipelines (e.g., STtools steps A1–A3 vs spacemake) on an identical input subset, recording measured times alongside published reference values to reproduce reported speedups.

## When to use

Use when two pipelines claim performance differences on the same data type and you need a fair head-to-head runtime comparison on a fixed FASTQ subset with identical references and comparable settings.

## When NOT to use

- You are evaluating analysis quality (clustering accuracy, spatial resolution) rather than computational runtime — use output-comparison skills instead.
- The pipelines process different input subsets or reference genomes, making wall-clock times non-comparable.
- Input is already a processed DGE/feature table; the benchmark requires raw FASTQ processing end-to-end.

## Inputs

- SRR14082757 subset raw FASTQ files (1st-seq spatial-barcode reads and 2nd-seq cDNA reads, 104.7G bases) from SRA
- Prebuilt mm10 STAR genome index
- STtools Step A1-generated spatial coordinates .txt files and error-corrected barcode whitelist.txt

## Outputs

- runtime_benchmark.csv with measured wall-clock runtimes per tool alongside reported values (254 min STtools vs 1100 min spacemake)
- STtools Step A3 digital expression matrix outputs (Gene, GeneFull, and Velocyto variants) confirming both pipelines completed on the identical subset

## How to apply

Fix a single input subset — here the SRR14082757 subset of SeqScope mouse liver data (GSM5212844, 104.7G bases, 1st-seq spatial-barcode reads and 2nd-seq cDNA reads) — and run both pipelines on exactly that subset with the same mm10 STAR genome index. For the STtools arm, run Step A1 with seqtk on the 1st-seq FASTQ to extract spatial coordinates (.txt) and generate the error-corrected whitelist.txt for STARsolo, then run Step A3 with STAR >=2.7.5c on the 2nd-seq FASTQ pair and whitelist to produce the Gene, GeneFull, and Velocyto DGE outputs. Run spacemake on the identical FASTQ subset with the mm10 reference under comparable settings. Record wall-clock runtime for each tool, write the comparison to runtime_benchmark.csv, and report measured values alongside the paper's reference benchmark (254 min for STtools vs 1100 min for spacemake, roughly four-fold faster).

## Related tools

- **STtools** (Pipeline under benchmark; run steps A1-A3 (seqtk coordinate extraction + STARsolo alignment) on the SRR14082757 subset) — https://github.com/seqscope/STtools
- **STARsolo** (Alignment engine invoked by STtools Step A3 to produce the DGE; requires STAR >=2.7.5c) — https://github.com/alexdobin/STAR
- **seqtk** (Used in Step A1 to process the 1st-seq FASTQ and extract spatial coordinates and the STARsolo whitelist) — https://github.com/lh3/seqtk
- **spacemake** (Comparison pipeline run on the identical FASTQ subset with the mm10 reference under comparable settings)

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq $STDATA/stepA_extractCoordinates/liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 $STDATA/stepA_align/liver_tile2106_sub_R1.fastq.gz --second-fq2 $STDATA/stepA_align/liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --star-path $STARPATH --seqtk-path $SEQTKPATH --seqscope1st 'HiSeq' --clustering False --lane-tiles 1_2106 --binsize 300 --window 150 -l 20 -o 'Sample' -c 2
```

## Evaluation signals

- Step A3 DGE output files (Gene, GeneFull, Velocyto variants) exist and are non-empty for both pipelines.
- Measured STtools runtime reproduces the reported several-fold speedup over spacemake (reported 254 min vs 1100 min).
- Both tools were run on the identical SRR14082757 FASTQ subset and the same mm10 STAR genome index.
- runtime_benchmark.csv contains measured values alongside the reported benchmark values.
- STAR version used is >=2.7.5c as required by STtools.

## Limitations

- The reported benchmark (254 min vs 1100 min) is for a subset (SRR14082757, 104.7G bases); runtime scaling to full SeqScope datasets is not established here.
- Runtime comparisons are hardware-dependent; measured times may deviate from the paper's values on different machines.
- 'Comparable settings' between pipelines is an approximation — exact parameter equivalence across STtools and spacemake is not guaranteed.

## Evidence

- [results] Benchmark reference values: "we ran both tools with the same mouse liver SeqScope data (GSM5212844)"
- [results] Subset scale: "we ran experiments for a subset (SRR14082757) with 104.7G bases"
- [methods] Step A3 definition: "**Step A3** takes valid barcodes `whitelist.txt`, 2nd-seq `fastq.gz` files, and the STAR indices of reference genome as input to run `STARsolo` alignment"
- [readme] DGE output: "this step outputs digital expression matrix (DGE)."
- [readme] STAR version requirement: "* STAR>=2.7.5c (Click for instructions to install [STAR](https://github.com/alexdobin/STAR))"
- [methods] STARsolo alignment role: "STtools aligns each cDNA sequence to the reference genome using STARsolo (Kaminow et al., 2021)."
