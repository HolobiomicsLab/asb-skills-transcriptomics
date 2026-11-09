---
name: runtime-latency-profiling
description: Use when comparing inference speed and memory of malignant cell annotation
  tools on scRNA-seq count matrices subsampled to 100–1,000,000 cells, on a fixed
  GPU/CPU hardware platform.
when_to_use_negative:
- The goal is annotation accuracy or generalization evaluation (e.g. the 95.16% average
  accuracy benchmark), not runtime or memory profiling
- Input data are hematologic tumors — the article states Cancer-Finder is not recommended
  for hematologic tumor data, so its runtime advantage is moot there
- The comparison requires end-to-end workflow time including model training rather
  than pre-trained-model inference latency
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Cancer-Finder
  role: Tool under benchmark; inference run via infer.py with the pre-trained checkpoint
    to measure direct-inference latency and memory
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: CaSee
  role: Comparison tool run on the GPU platform with default parameters to record
    total time between input reading and output
  repo: https://github.com/yuansh3354/CaSee
- name: CopyKAT
  role: CNV-based comparison tool run on the CPU platform with default parameters;
    batches data above 10,000 cells as its documentation recommends
  repo: https://github.com/navinlabcode/copykat
- name: SCEVAN
  role: Clonal-structure-based comparison tool run on the CPU platform with default
    parameters to record time and peak memory
  repo: https://github.com/AntonioDeFalco/SCEVAN
- name: ikarus
  role: Machine-learning comparison tool run on the CPU platform with default parameters
    to record time and peak memory
  repo: https://github.com/BIMSBbioinfo/ikarus
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:281cca9c5d14b80024358395ff91cf7cae628dc545383290a964e1c6ae2a13ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - CaSee
  - Cancer-Finder
  - CopyKAT
  - SCEVAN
  - ikarus
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Runtime latency profiling

## Summary

Benchmarks wall-clock inference time and memory consumption of malignant cell annotation tools (Cancer-Finder, CaSee, CopyKAT, SCEVAN, ikarus) on scRNA-seq count matrices subsampled from 100 to 1,000,000 cells, to demonstrate that direct inference with a pre-trained domain generalization model scales better than CNV- or clonal-structure-based methods.

## When to use

Use when comparing inference speed and memory of malignant cell annotation tools on scRNA-seq count matrices subsampled to 100–1,000,000 cells, on a fixed GPU/CPU hardware platform.

## When NOT to use

- The goal is annotation accuracy or generalization evaluation (e.g. the 95.16% average accuracy benchmark), not runtime or memory profiling
- Input data are hematologic tumors — the article states Cancer-Finder is not recommended for hematologic tumor data, so its runtime advantage is moot there
- The comparison requires end-to-end workflow time including model training rather than pre-trained-model inference latency

## Inputs

- scRNA-seq raw UMI count matrices (tsv/csv) subsampled to 100, 1,000, 10,000, 100,000 and 1,000,000 cells
- Binary-stored count matrices for the 10,000- and 100,000-cell parsing-overhead runs
- Pre-trained Cancer-Finder checkpoint: checkpoints/sc_pretrain_article.pkl
- Benchmark hardware: Intel Xeon Silver 4216 @ 2.10 GHz (16 cores), NVIDIA RTX 3090, 512 GB RAM

## Outputs

- Per-size wall-clock inference times for each tool (Cancer-Finder: 1.39 s, 2.53 s, 58.65 s, 3,903.41 s, 38,903.92 s)
- Binary-vs-csv parsing overhead estimates (~4.15 s at 10k cells, ~39.46 s at 100k cells)
- Peak memory consumption per tool and dataset size
- Combined results table: inference_benchmark.csv

## How to apply

Load the pre-trained Cancer-Finder checkpoint (checkpoints/sc_pretrain_article.pkl) and subsample input count matrices to 100, 1,000, 10,000, 100,000 and 1,000,000 cells in tsv/csv format accepted by infer.py. Run Cancer-Finder with infer.py on the GPU platform (Intel Xeon Silver 4216 @ 2.10 GHz, 16 cores, NVIDIA RTX 3090, 512 GB RAM), recording wall-clock time from data reading to output; published times are 1.39 s, 2.53 s, 58.65 s, 3,903.41 s and 38,903.92 s. Repeat the 10,000- and 100,000-cell runs with binary-stored matrices to isolate csv-parsing overhead (expected ~4.15 s and ~39.46 s). Run CaSee on the GPU and CopyKAT, SCEVAN and ikarus on the CPU with default parameters, recording total time and peak memory; batch data above 10,000 cells for CopyKAT as its documentation recommends. Tabulate per-size times and memory for all five tools and save as inference_benchmark.csv, confirming Cancer-Finder is fastest with significantly reduced memory (Fig. 3c,d).

## Related tools

- **Cancer-Finder** (Tool under benchmark; inference run via infer.py with the pre-trained checkpoint to measure direct-inference latency and memory) — https://github.com/Patchouli-M/SequencingCancerFinder
- **CaSee** (Comparison tool run on the GPU platform with default parameters to record total time between input reading and output) — https://github.com/yuansh3354/CaSee
- **CopyKAT** (CNV-based comparison tool run on the CPU platform with default parameters; batches data above 10,000 cells as its documentation recommends) — https://github.com/navinlabcode/copykat
- **SCEVAN** (Clonal-structure-based comparison tool run on the CPU platform with default parameters to record time and peak memory) — https://github.com/AntonioDeFalco/SCEVAN
- **ikarus** (Machine-learning comparison tool run on the CPU platform with default parameters to record time and peak memory) — https://github.com/BIMSBbioinfo/ikarus

## Examples

```
python infer.py --checkpoint checkpoints/sc_pretrain_article.pkl --input subset_10000.csv
```

## Evaluation signals

- Cancer-Finder timings fall on the order of 1.39 s, 2.53 s, 58.65 s, 3,903.41 s and 38,903.92 s for 100, 1,000, 10,000, 100,000 and 1,000,000 cells respectively
- Binary-matrix re-runs of the 10,000- and 100,000-cell subsets yield ~4.15 s and ~39.46 s, quantifying csv-parsing overhead
- inference_benchmark.csv contains per-size inference time and memory consumption for all five tools
- Cancer-Finder shows the fastest times and significantly reduced memory consumption, matching Fig. 3c,d
- CopyKAT runs on datasets above 10,000 cells are processed in batches consistent with its documentation

## Limitations

- csv parsing dominates wall-clock time at smaller dataset sizes (~4.15 s vs 58.65 s at 10k cells; ~39.46 s vs 3,903.41 s at 100k cells), so input format strongly affects reported latency
- Results are hardware-specific (Intel Xeon Silver 4216, NVIDIA RTX 3090, 512 GB RAM) and cross-platform (GPU vs CPU) comparisons do not isolate algorithmic speed alone
- CopyKAT cannot run on datasets above 10,000 cells without batching, and its README suggests parallelizing individual samples and combining outputs, which complicates end-to-end timing
- These findings are timing-specific and do not establish annotation accuracy, which the article reports separately (95.16% average accuracy)

## Evidence

- [results] Published Cancer-Finder inference timings across five dataset sizes: [evidence span withheld — anchor status: not_found]
- [abstract] Cancer-Finder's design goal of rapid malignant cell identification: "we present Cancer-Finder, a domain generalization-based deep-learning algorithm that can rapidly identify malignant cells in single-cell data"
- [discussion] Rationale for why CNV-based comparison tools are slower: "CopyKAT must infer CNV and classify based on CNV profiles"
- [readme] CopyKAT README guidance on handling multiple/large samples that informs batching during benchmarking: [evidence span withheld — anchor status: external_doc]
