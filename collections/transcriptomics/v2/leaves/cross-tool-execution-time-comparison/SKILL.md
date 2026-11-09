---
name: cross-tool-execution-time-comparison
description: Use when comparing computational efficiency (CPU time and wall-clock
  time) of multiple pathogen-quantification pipelines for scRNAseq data, and a measured
  ranking must be reproduced on an identical input sample.
when_to_use_negative:
- The goal is accuracy or sensitivity comparison of pathogen read detection rather
  than runtime cost
- The tools cannot be run on the same input sample with identical thread allocation,
  so measured runtimes would not be comparable
- Input is a single tool's run and no cross-tool benchmark is needed
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: scPathoQuant
  role: Tool under test; quantification step wrapped in `time` with default bbmap
    aligner and -p 8
  repo: https://github.com/galelab/scPathoQuant
- name: Python
  role: Language of scPathoQuant and the benchmark scripts (PathogenTrack.py count,
    module_detection.py)
- name: docker
  role: Alternative containerized execution mode for running scpathoquant in the benchmark
  repo: https://github.com/galelab/scPathoQuant
- name: ViralTrack
  role: Comparison tool; Viral_Track_scanning.R run under `time` with -p 8 on the
    same sample
  repo: https://github.com/PierreBSC/Viral-Track
- name: PathogenTrack
  role: Comparison tool; PathogenTrack.py count run under `time` with -p 8 on the
    same sample
  repo: https://github.com/ncrna/PathogenTrack
- name: Venus
  role: Comparison tool; module_detection.py run under `time` with -p 8 on the same
    sample
  repo: https://github.com/aicb-ZhangLabs/Venus
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:5d50c111240ad705789bd6988b27e8e0896f4a549322187267591f6b1bf58824
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-tool-execution-time-comparison

## Summary

Benchmark the CPU-time cost of scPathoQuant's quantification step against ViralTrack, PathogenTrack, and Venus by wrapping each tool in the Linux `time` command on the same 10x scRNAseq sample and ranking fastest-to-slowest.

## When to use

Use when comparing computational efficiency (CPU time and wall-clock time) of multiple pathogen-quantification pipelines for scRNAseq data, and a measured ranking must be reproduced on an identical input sample.

## When NOT to use

- The goal is accuracy or sensitivity comparison of pathogen read detection rather than runtime cost
- The tools cannot be run on the same input sample with identical thread allocation, so measured runtimes would not be comparable
- Input is a single tool's run and no cross-tool benchmark is needed

## Inputs

- HIV infected donor 2 scRNAseq 10x cellranger output sample (from Bradley et al. test set, via galelab/Whitmore_scPathoQuant_testSets)
- Pathogen reference genome folder containing pathogen FASTA (and optional GTF)
- Benchmark scripts: CellRangerwithSPQBench.sh, ViralTrackBench.sh, PathogenTrackBench.sh, VenusBench.sh

## Outputs

- timing_table.csv with CPU time and wall-clock time per tool's quantification step
- Fastest-to-slowest tool ranking (expected: scPathoQuant < ViralTrack < PathogenTrack < Venus)

## How to apply

Obtain the HIV infected donor 2 test dataset (10x cellranger output plus a pathogen reference genome folder) and the comparison scripts from the galelab/Whitmore_scPathoQuant_testSets repository (CellRangerwithSPQBench.sh, ViralTrackBench.sh, PathogenTrackBench.sh, VenusBench.sh). Run each tool's quantification step under the Linux `time` command with the same input sample and identical thread allocation (-p 8): scpathoquant with the default bbmap aligner, Viral_Track_scanning.R, PathogenTrack.py count, and module_detection.py (Venus). Record CPU time and wall-clock time for each tool and rank them fastest to slowest, writing the results to timing_table.csv. Validation requires reproducing the reported benchmark ordering scPathoQuant < ViralTrack < PathogenTrack < Venus (Fig. 5E), with all four tools completing on the same input.

## Related tools

- **scPathoQuant** (Tool under test; quantification step wrapped in `time` with default bbmap aligner and -p 8) — https://github.com/galelab/scPathoQuant
- **Python** (Language of scPathoQuant and the benchmark scripts (PathogenTrack.py count, module_detection.py))
- **docker** (Alternative containerized execution mode for running scpathoquant in the benchmark) — https://github.com/galelab/scPathoQuant
- **ViralTrack** (Comparison tool; Viral_Track_scanning.R run under `time` with -p 8 on the same sample) — https://github.com/PierreBSC/Viral-Track
- **PathogenTrack** (Comparison tool; PathogenTrack.py count run under `time` with -p 8 on the same sample) — https://github.com/ncrna/PathogenTrack
- **Venus** (Comparison tool; module_detection.py run under `time` with -p 8 on the same sample) — https://github.com/aicb-ZhangLabs/Venus

## Examples

```
time scpathoquant -10x <sample> -op <results> -p 8 -p2genome <pathogen_folder>
```

## Evaluation signals

- Measured runtime ordering reproduces scPathoQuant < ViralTrack < PathogenTrack < Venus as reported in Fig. 5E
- All four tools complete successfully on the same HIV donor 2 input sample
- CPU time and wall-clock time are recorded for every tool's quantification step and written to timing_table.csv
- Each tool was run with identical thread allocation (-p 8) on the same input

## Limitations

- Ranking was validated on a single sample (HIV infected donor 2) and a fixed thread allocation (-p 8); other datasets or thread counts may change the ordering
- Only the quantification step timing is compared, not full end-to-end pipeline cost including cellranger count
- Large _tmp/ temporary files produced by scpathoquant may consume disk space unless --tmp_removal is specified

## Evidence

- [results] benchmark ranking: [evidence span withheld — anchor status: not_found]
- [readme] benchmark commands: [evidence span withheld — anchor status: external_doc]
- [readme] thread parameter: "* -p = number of processors (defualt = 1)"
- [readme] test dataset location: "Examples and test data sets and codes for scPathoQuant can be found [here](https://github.com/galelab/Whitmore_scPathoQuant_testSets)"
- [readme] disk caveat: "--tmp_removal = if specified will remove the temporary directory (_tmp/) of files used by scpathoquant (these files can be large so if space is an issue these should be deleted)"
