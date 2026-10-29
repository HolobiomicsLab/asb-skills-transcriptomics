---
name: high-memory-genome-alignment
description: Use when QC-filtered small RNA-seq FASTQ reads (e.g., 17–50 nt post-trimming
  from 50 nt serum reads) must be mapped to hg19/hg38 with STAR under a ~30 GB RAM
  budget, or when per-sample runtime must be compared against web-based pipelines
  like exceRpt run with identical parameters.
when_to_use_negative:
- Input reads are already aligned (COMPSRA accepts fastq or SAM, but the runtime benchmark
  and QC→alignment workflow assume raw FASTQ)
- The server has far less than ~30 GB RAM or cannot build the STAR hg38 index (first-run
  indexing costs about 3 hours)
- A web-based upload pipeline is required for small data — the article notes all web
  tools 'need users to upload the original sequencing files, which is un-workable
  for big data', which is the scenario COMPSRA's stand-alone alignment avoids
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: COMPSRA
  role: Stand-alone Java platform whose Alignment module wraps STAR to map qualified
    reads to hg19/hg38 and time per-sample runtime
  repo: https://github.com/cougarlj/COMPSRA
- name: STAR
  role: Default RNA-sequence aligner used by COMPSRA's Alignment module with default,
    command-line-customizable parameters
- name: exceRpt
  role: Comparable web-based pipeline run with the same parameters on the same 12
    serum FASTQ inputs to provide the ~20 min/sample runtime baseline
- name: Java
  role: Runtime environment (JRE 8 or up) required to execute COMPSRA v1.0
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:5eeaa62b673b26a052e0bdc9d11ed9a48cf589910243438cc20a9e061910ad3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - Java
  - STAR
  - exceRpt
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# high-memory-genome-alignment

## Summary

Maps QC-filtered small RNA-seq reads to the human genome (hg19/hg38) with STAR inside the COMPSRA platform on a 30 GB RAM server, benchmarked at ~10 minutes per 50 nt serum sample versus ~20 minutes per sample for the web-based exceRpt pipeline.

## When to use

Use when QC-filtered small RNA-seq FASTQ reads (e.g., 17–50 nt post-trimming from 50 nt serum reads) must be mapped to hg19/hg38 with STAR under a ~30 GB RAM budget, or when per-sample runtime must be compared against web-based pipelines like exceRpt run with identical parameters.

## When NOT to use

- Input reads are already aligned (COMPSRA accepts fastq or SAM, but the runtime benchmark and QC→alignment workflow assume raw FASTQ)
- The server has far less than ~30 GB RAM or cannot build the STAR hg38 index (first-run indexing costs about 3 hours)
- A web-based upload pipeline is required for small data — the article notes all web tools 'need users to upload the original sequencing files, which is un-workable for big data', which is the scenario COMPSRA's stand-alone alignment avoids

## Inputs

- QC-filtered small RNA-seq FASTQ files (default input; original raw reads were 50 nt, trimmed to 0–42 nt after adapter and random-base removal)
- STAR reference genome index for hg19/hg38 (pre-built or downloaded via COMPSRA ToolKit)
- A sample list file (-inf) with one FASTQ path per line for multi-sample runs

## Outputs

- STAR alignment output (e.g., sample01_17to50_FitRead_STAR_Aligned.out.bam) per sample
- FASTA/BAM of reads unmapped to the human genome, reserved for the Microbe module
- Per-sample runtime measurements (~10 min COMPSRA vs ~20 min exceRpt) aggregated e.g. into runtime_benchmark.csv

## How to apply

First pass reads through COMPSRA's Quality Control module (adapter trimming with -ra TGGAATTCTCGGGTGCCAAGG, removal of 4 random bases at both ends via -rb 4, quality filters -rh 20 -rt 20 -rr 20, and read-length grouping -rlh 8,17), then feed the qualified FASTQ into the Alignment module with -aln -mt star. Run STAR with default (customizable) parameters against the pre-built hg38 index on a server with 30 GB RAM, mirroring the parameters used for exceRpt so runtimes are comparable; the STAR genome index can be fetched and built with the COMPSRA ToolKit (java -jar COMPSRA.jar -tk -dr -ck star_hg38). Pass -mbi on the first run so COMPSRA builds STAR index files (about 3 hours). Aligned reads flow to the Annotation module while reads unmapped to the human genome are saved into a FASTA/BAM file for the Microbe module (BLAST against the UCSC nt database). Validate by timing the full QC→STAR→annotation pipeline per sample: expected ~10 minutes per sample for COMPSRA versus ~20 minutes per sample for exceRpt on the same 12 serum FASTQ inputs.

## Related tools

- **COMPSRA** (Stand-alone Java platform whose Alignment module wraps STAR to map qualified reads to hg19/hg38 and time per-sample runtime) — https://github.com/cougarlj/COMPSRA
- **STAR** (Default RNA-sequence aligner used by COMPSRA's Alignment module with default, command-line-customizable parameters)
- **exceRpt** (Comparable web-based pipeline run with the same parameters on the same 12 serum FASTQ inputs to provide the ~20 min/sample runtime baseline)
- **Java** (Runtime environment (JRE 8 or up) required to execute COMPSRA v1.0)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -aln -mt star -mbi -in ./example_out/sample01/sample01_17to50_FitRead.fastq.gz -out ./example_out/sample01/sample01_17to50_FitRead
```

## Evaluation signals

- Per-sample pipeline runtime approximates the reported ~10 minutes for COMPSRA and ~20 minutes for exceRpt on the 12 serum samples
- STAR alignment was run against hg38 on a server with 30 GB RAM, using parameters identical to those set for exceRpt
- Output BAM/alignment files follow the expected naming pattern (e.g., <sample>_17to50_FitRead_STAR_Aligned.out.bam) and an unmapped-read file exists for each sample for the Microbe module
- The first alignment run completed the STAR index build (about 3 hours) or used a pre-built index; subsequent runs skip -mbi
- Microbe module runtime averaged ~3.5 hours across the 12 samples for BLAST against the UCSC nt database over viruses, archaea, bacteria and fungi

## Limitations

- The exact COMPSRA software version used for the runtime comparison is not stated in the article's results (the README documents v1.0, with patches up to V1.0.3); runtimes may differ across versions
- The first alignment run requires building STAR index files, which costs about 3 hours, and index building can fail silently unless the STAR binary is made executable (chmod)
- Runtime figures (~10 min vs ~20 min per sample) are tied to the specific 30 GB RAM server and 12 human serum samples (50 nt reads); they do not generalize to other hardware or data sizes
- COMPSRA supports only hg19/hg38 for human alignment (plus rat GCA_000001895.4/rno6 and .3/rno5 for miRNA/piRNA prebuilt databases)

## Evidence

- [results] Runtime benchmark on 30 GB RAM server with matched parameters: "We run COMPSRA on server with 30 GB RAM and set the same parameters as exceRpt"
- [results] Per-sample runtime comparison: "COMPSRA takes ~10 minutes per sample, but exceRpt will cost ~20 minutes for each sample (see Supplementary Fig. S1)"
- [methods] Alignment module behavior: "Qualified reads from the QC module output are first mapped to the target genome (e.g., hg19/hg38), and then aligned reads are quantified and annotated in the Annotation Module"
- [readme] Unmapped reads routed to Microbe module: "Reads that could not be mapped to the human genome are saved into a FASTA file for input into the Microbe Module."
- [readme] First-run STAR index build cost: "(Note: -mbi was only needed for the first time when you run COMPSRA and built the index files for STAR. This process will cost about 3 hours.)"
