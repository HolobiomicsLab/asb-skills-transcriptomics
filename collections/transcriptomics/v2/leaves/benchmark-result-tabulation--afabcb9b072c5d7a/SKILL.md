---
name: benchmark-result-tabulation
description: Use when comparing the computational cost (CPU time and wall-clock time)
  of pathogen-quantification tools for 10x single-cell data on one identical sample,
  to produce a ranked runtime table.
when_to_use_negative:
- The goal is quantifying pathogen read/UMI counts or accuracy rather than measuring
  computational cost — use the tools' own quantification skills instead
- The input is raw FASTQ rather than 10x cellranger output; this benchmark runs each
  tool on the cellranger-processed sample
- Only a single tool needs to be run — this skill requires running all four tools
  (scPathoQuant, ViralTrack, PathogenTrack, Venus) on the identical input for a ranking
tools:
- name: scPathoQuant
  role: Tool under benchmark; its quantification step (default bbmap aligner) is timed
    with the Linux time command
  repo: https://github.com/galelab/scPathoQuant
- name: ViralTrack
  role: Comparison tool; Viral_Track_scanning.R is timed under `time` with the same
    sample and thread allocation
  repo: https://github.com/PierreBSC/Viral-Track
- name: PathogenTrack
  role: Comparison tool; PathogenTrack.py count is timed under `time` with the same
    sample and thread allocation
  repo: https://github.com/ncrna/PathogenTrack
- name: Venus
  role: Comparison tool; module_detection.py is timed under `time` with the same sample
    and thread allocation
  repo: https://github.com/aicb-ZhangLabs/Venus
- name: docker
  role: Alternative containerized execution environment for running scPathoQuant in
    the benchmark
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:caf93c99457f5f5fe469f4c74acb12806620b0b04197da3c839253bd46a64b4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - PathogenTrack
  - Python
  - Venus
  - ViralTrack
  - docker
  - scPathoQuant
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# benchmark-result-tabulation

## Summary

Reproduce the scPathoQuant paper's CPU-time benchmark ranking by timing the quantification step of scPathoQuant (SPQ), ViralTrack, PathogenTrack, and Venus on the same 10x scRNAseq sample and tabulating the fastest-to-slowest ordering.

## When to use

Use when comparing the computational cost (CPU time and wall-clock time) of pathogen-quantification tools for 10x single-cell data on one identical sample, to produce a ranked runtime table.

## When NOT to use

- The goal is quantifying pathogen read/UMI counts or accuracy rather than measuring computational cost — use the tools' own quantification skills instead
- The input is raw FASTQ rather than 10x cellranger output; this benchmark runs each tool on the cellranger-processed sample
- Only a single tool needs to be run — this skill requires running all four tools (scPathoQuant, ViralTrack, PathogenTrack, Venus) on the identical input for a ranking

## Inputs

- 10x cellranger output folder for the HIV infected donor 2 scRNAseq sample (filtered/raw_feature_bc_matrix)
- Pathogen reference genome folder (HIV fasta file, optionally with a pathogen gtf file)
- Benchmark scripts from the galelab/Whitmore_scPathoQuant_testSets repository (CellRangerwithSPQBench.sh, ViralTrackBench.sh, PathogenTrackBench.sh, VenusBench.sh)
- Comparison tool scripts: Viral_Track_scanning.R (ViralTrack), PathogenTrack.py count (PathogenTrack), module_detection.py (Venus)

## Outputs

- timing_table.csv ranking the four tools' quantification steps from fastest to slowest
- Per-tool CPU time and wall-clock time measurements for the quantification step on the HIV donor 2 sample

## How to apply

Obtain the HIV infected donor 2 test dataset (10x cellranger output plus a pathogen reference genome folder containing the HIV fasta, optionally a gtf) and the benchmark scripts (CellRangerwithSPQBench.sh, ViralTrackBench.sh, PathogenTrackBench.sh, VenusBench.sh) from the galelab/Whitmore_scPathoQuant_testSets repository. Run each tool's quantification step under the Linux `time` command with identical input and thread allocation (-p 8): scpathoquant with its default bbmap aligner, Viral_Track_scanning.R, PathogenTrack.py count, and Venus's module_detection.py. Record CPU time and wall-clock time for each tool, rank the tools from fastest to slowest, and write the comparison to timing_table.csv. Correctness is judged by whether the measured ordering reproduces the reported benchmark ranking SPQ < ViralTrack < PathogenTrack < Venus (Fig. 5E).

## Related tools

- **scPathoQuant** (Tool under benchmark; its quantification step (default bbmap aligner) is timed with the Linux time command) — https://github.com/galelab/scPathoQuant
- **ViralTrack** (Comparison tool; Viral_Track_scanning.R is timed under `time` with the same sample and thread allocation) — https://github.com/PierreBSC/Viral-Track
- **PathogenTrack** (Comparison tool; PathogenTrack.py count is timed under `time` with the same sample and thread allocation) — https://github.com/ncrna/PathogenTrack
- **Venus** (Comparison tool; module_detection.py is timed under `time` with the same sample and thread allocation) — https://github.com/aicb-ZhangLabs/Venus
- **docker** (Alternative containerized execution environment for running scPathoQuant in the benchmark)

## Examples

```
time scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/pathogen/fastafilefolder
```

## Evaluation signals

- Measured runtime ordering reproduces the reported benchmark ranking scPathoQuant < ViralTrack < PathogenTrack < Venus (Fig. 5E)
- All four tools complete successfully on the same HIV donor 2 input sample
- Both CPU time and wall-clock time are recorded for each tool's quantification step
- All tools are given the same thread allocation (-p 8) and the same 10x sample input, so timing differences are attributable to the tools
- timing_table.csv is produced with a complete fastest-to-slowest ranking of the four tools

## Limitations

- The benchmark is demonstrated on a single sample (HIV infected donor 2 scRNAseq), so the ranking may not generalize to other datasets or pathogens
- scPathoQuant timing reflects its default bbmap aligner; using -align bowtie2 with different parameters would change runtime
- Timings depend on the fixed thread allocation (-p 8); different hardware or thread counts can alter the ordering
- Venus requires running module_detection.py rather than a single monolithic command, so 'quantification step' boundaries differ across tools

## Evidence

- [readme] Here we provide codes used for benchmarking scPathoQuant against ['ViralTrack](https://github.com/PierreBSC/Viral-Track), [Venus](https://github.com/aicb-ZhangLabs/Venus): [evidence span withheld — anchor status: not_found]
- [results] SPQ was the fastest tool, requiring the least CPU time for its quantification step, followed by ViralTrack, then PathogenTrack, with Venus being the most computationally intensive (Fig. 5E).: [evidence span withheld — anchor status: not_found]
- [methods] Examples and test data sets and codes for scPathoQuant can be found [here](https://github.com/galelab/Whitmore_scPathoQuant_testSets): "Examples and test data sets and codes for scPathoQuant can be found [here](https://github.com/galelab/Whitmore_scPathoQuant_testSets)"
- [intro] scPathoQuant then takes the unaligned reads from the host genome and maps them to your pathogen genome of interest and then integrates them back into the cellranger files: "scPathoQuant then takes the unaligned reads from the host genome and maps them to your pathogen genome of interest and then integrates them back into the cellranger files"
