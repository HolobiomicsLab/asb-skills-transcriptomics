---
name: single-cell-pathogen-quantification-benchmarking
description: Use when comparing the computational cost (CPU/wall-clock time) of pathogen
  quantification tools—scPathoQuant, ViralTrack, PathogenTrack, Venus—on an identical
  10x scRNAseq sample to rank them from fastest to slowest.
when_to_use_negative:
- The goal is comparing quantification accuracy or sensitivity rather than computational
  runtime — this skill measures only CPU/wall-clock cost
- The input is not a 10x cellranger output sample with an accompanying pathogen reference
  genome folder, since the four tools must run on the identical input for timing to
  be comparable
- A single tool's runtime is needed on its own data; timing comparisons require all
  four tools (ViralTrack, PathogenTrack, Venus, scPathoQuant) run under identical
  thread allocation on the same sample
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: scPathoQuant
  role: Tool under test; its quantification step (default bbmap aligner) is timed
    as the reference, expected to be the fastest
  repo: https://github.com/galelab/scPathoQuant
- name: ViralTrack
  role: Comparison tool; Viral_Track_scanning.R is timed under `time` and expected
    to be second fastest after scPathoQuant
  repo: https://github.com/PierreBSC/Viral-Track
- name: PathogenTrack
  role: Comparison tool; PathogenTrack.py count is timed under `time` and expected
    to be slower than scPathoQuant and ViralTrack
  repo: https://github.com/ncrna/PathogenTrack
- name: Venus
  role: Comparison tool; module_detection.py is timed under `time` and expected to
    be the most computationally intensive
  repo: https://github.com/aicb-ZhangLabs/Venus
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:fd1399246089355b20bc6be3ecf9036dc5728ec34ab2784b9a64e600c9282b07
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
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# single-cell-pathogen-quantification-benchmarking

## Summary

Benchmark the CPU time of scPathoQuant's quantification step against ViralTrack, PathogenTrack, and Venus on the same 10x scRNAseq sample (HIV infected donor 2) to reproduce the reported runtime ranking from the scPathoQuant paper (Fig. 5E).

## When to use

Use when comparing the computational cost (CPU/wall-clock time) of pathogen quantification tools—scPathoQuant, ViralTrack, PathogenTrack, Venus—on an identical 10x scRNAseq sample to rank them from fastest to slowest.

## When NOT to use

- The goal is comparing quantification accuracy or sensitivity rather than computational runtime — this skill measures only CPU/wall-clock cost
- The input is not a 10x cellranger output sample with an accompanying pathogen reference genome folder, since the four tools must run on the identical input for timing to be comparable
- A single tool's runtime is needed on its own data; timing comparisons require all four tools (ViralTrack, PathogenTrack, Venus, scPathoQuant) run under identical thread allocation on the same sample

## Inputs

- HIV infected donor 2 10x scRNAseq sample (cellranger count output folder, e.g. filtered_feature_bc_matrix with features.tsv.gz and matrix.mtx.gz)
- Pathogen reference genome folder with at most two files: a fasta file (e.g. HIV genome with simplified header such as > HIV_virus) and an optional pathogen gtf file
- Benchmark scripts from galelab/Whitmore_scPathoQuant_testSets (CellRangerwithSPQBench.sh, ViralTrackBench.sh, PathogenTrackBench.sh, VenusBench.sh)

## Outputs

- timing_table.csv ranking the four tools' quantification steps from fastest to slowest by CPU time
- Per-tool CPU time and wall-clock time measurements on the shared HIV donor 2 input
- Reproduction of the reported benchmark ordering scPathoQuant < ViralTrack < PathogenTrack < Venus (Fig. 5E)

## How to apply

Obtain the HIV infected donor 2 test dataset (10x cellranger count output plus a pathogen reference genome folder containing the fasta and optional gtf) and the benchmark scripts from the galelab/Whitmore_scPathoQuant_testSets repository (CellRangerwithSPQBench.sh, ViralTrackBench.sh, PathogenTrackBench.sh, VenusBench.sh). Run each tool's quantification step under the Linux `time` command on the same input sample with the same thread allocation (-p 8): scpathoquant with the default bbmap aligner, Viral_Track_scanning.R, PathogenTrack.py count, and module_detection.py (Venus). Record both CPU time and wall-clock time for each tool, rank the tools fastest to slowest, and write the ranking to timing_table.csv. Validate by checking that the measured ordering reproduces scPathoQuant < ViralTrack < PathogenTrack < Venus, with all four tools completing on the identical input.

## Related tools

- **scPathoQuant** (Tool under test; its quantification step (default bbmap aligner) is timed as the reference, expected to be the fastest) — https://github.com/galelab/scPathoQuant
- **ViralTrack** (Comparison tool; Viral_Track_scanning.R is timed under `time` and expected to be second fastest after scPathoQuant) — https://github.com/PierreBSC/Viral-Track
- **PathogenTrack** (Comparison tool; PathogenTrack.py count is timed under `time` and expected to be slower than scPathoQuant and ViralTrack) — https://github.com/ncrna/PathogenTrack
- **Venus** (Comparison tool; module_detection.py is timed under `time` and expected to be the most computationally intensive) — https://github.com/aicb-ZhangLabs/Venus

## Examples

```
time scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/pathogen/fastafilefolder
```

## Evaluation signals

- The measured CPU-time ordering must reproduce the reported ranking scPathoQuant < ViralTrack < PathogenTrack < Venus (Fig. 5E)
- All four tools must complete successfully on the same HIV infected donor 2 scRNAseq input sample
- Every tool is run with identical thread allocation (-p 8) and under the Linux `time` command so timings are comparable
- Both CPU time and wall-clock time are recorded per tool and written to timing_table.csv
- Benchmark scripts used match those published in Whitmore_scPathoQuant_testSets (CellRangerwithSPQBench.sh, ViralTrackBench.sh, PathogenTrackBench.sh, VenusBench.sh)

## Limitations

- The benchmark is defined on a single test sample (HIV infected donor 2 from the Bradley et al. dataset); rankings may differ on other pathogen/host combinations
- scPathoQuant timing assumes the default bbmap aligner; using -align bowtie2 with --very-sensitive --non-deterministic parameters would change runtime characteristics
- scPathoQuant leaves a temporary _tmp/ directory of large files unless --tmp_removal is specified, which affects disk but not CPU-time ranking
- Installations differ per tool (scPathoQuant via conda env create -f condaenv.yml or docker build -t scpathoquant); environment setup overhead is not part of the quantification-step timing

## Evidence

- [readme] scPathoQuant then takes the unaligned reads from the host genome and maps them to your pathogen genome of interest and then integrates them back into the cellranger files.: "scPathoQuant then takes the unaligned reads from the host genome and maps them to your pathogen genome of interest and then integrates them back into the cellranger files."
- [readme] Here we provide codes used for benchmarking scPathoQuant against 'ViralTrack, Venus and PathogenTrack: [evidence span withheld — anchor status: not_found]
- [intro] The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads.: "The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads."
- [methods] -10x = Path/to/10x/sample/ (this the path to the output folder generated by first running cellranger count ): "-10x = Path/to/10x/sample/ (this the path to the output folder generated by first running cellranger count )"
- [other] conda env create -f condaenv.yml -n scpathoquant: "conda env create -f condaenv.yml -n scpathoquant"
