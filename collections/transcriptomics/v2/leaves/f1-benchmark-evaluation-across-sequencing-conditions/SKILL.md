---
name: f1-benchmark-evaluation-across-sequencing-conditions
description: Use when a detector's calls (e.g. ecDNA callers on ATAC-Seq data) must
  be validated against known ground truth, especially to characterize performance
  sensitivity to local sequencing depth (e.g. 10x–30x+) and read length (e.g. 35–150
  bp).
when_to_use_negative:
- Input is real tumor data without ground-truth ecDNA coordinates — accuracy cannot
  be scored, only discovery performed
- Target detector is meant for WGS-based focal amplification detection — the article's
  benchmarking strategy is ATAC-Seq-specific and cannot guarantee accuracy for all
  types of focal amplifications
- Libraries have local depth below 10x — the benchmark itself shows circlehunter does
  not outperform Circle_finder in this regime
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: circlehunter
  role: Primary ecDNA detector under evaluation; its Snakemake pipeline processes
    simulated ATAC-Seq libraries and its F1 scores are the benchmark output
  repo: https://github.com/suda-huanglab/circlehunter
- name: Circle_finder
  role: Comparator detector that collects junction reads from supplementary alignments;
    called on every simulated library to benchmark against circlehunter
- name: ART
  role: Simulator used to generate sequencing reads for 500 mock ecDNAs sampled from
    GRCh38
- name: Snakemake
  role: Workflow engine that runs the circlehunter pipeline on all simulated samples
    via the Snakefile and config.yaml
- name: ATAC-Seq
  role: Assay whose enhanced chromatin accessibility signal on ecDNA underpins detection
    and provides background reads for simulated libraries
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:60e0f477cdba870355e7657828bf89b062a77b15961ff356a80ac750f22b818e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - ART
  - ATAC-Seq
  - Circle_finder
  - Snakemake
  - circlehunter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# f1-benchmark-evaluation-across-sequencing-conditions

## Summary

Benchmark ecDNA detection accuracy by sequencing ecDNA calls from competing tools (circlehunter vs Circle_finder) against simulated ground truth, computing precision, recall, and F1 across varying local depth and read-length conditions.

## When to use

Use when a detector's calls (e.g. ecDNA callers on ATAC-Seq data) must be validated against known ground truth, especially to characterize performance sensitivity to local sequencing depth (e.g. 10x–30x+) and read length (e.g. 35–150 bp).

## When NOT to use

- Input is real tumor data without ground-truth ecDNA coordinates — accuracy cannot be scored, only discovery performed
- Target detector is meant for WGS-based focal amplification detection — the article's benchmarking strategy is ATAC-Seq-specific and cannot guarantee accuracy for all types of focal amplifications
- Libraries have local depth below 10x — the benchmark itself shows circlehunter does not outperform Circle_finder in this regime

## Inputs

- Simulated ground-truth ecDNA coordinates (500 mock ecDNAs from GRCh38, 5 kb-10 Mb, 1-50 segments each)
- Simulated sequencing reads generated with ART at varying read lengths and local depths
- Real background ATAC-Seq reads (GM12878, ENCODE GSE170245) for blending
- Detector calls from circlehunter and Circle_finder on each simulated library
- YAML config enumerating samples and outputs (config.yaml, copied from config-template.yaml)

## Outputs

- Per-condition precision, recall, and F1 tables across local depth and read length
- Depth-sweep F1 table (Fig. 2A) and read-length-sweep F1 table (Fig. 2B), e.g. f1_scores_fig2ab.csv

## How to apply

Simulate ground-truth libraries with ART — for circlehunter, 500 mock ecDNAs sampled from GRCh38, each 5 kb-10 Mb in size and composed of 1-50 genomic segments — then blend their reads with real ATAC-Seq background reads (GM12878, ENCODE GSE170245) to sweep local depths and read lengths. Configure all simulated samples and outputs in the circlehunter YAML config and run the Snakemake pipeline (`snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml`). Run each detector (circlehunter and Circle_finder) on every simulated library, then match each tool's called ecDNA coordinates to the simulated ground truth to compute per-condition precision, recall, and F1. Assemble the depth-sweep and read-length-sweep F1 tables (e.g. f1_scores_fig2ab.csv) and judge success against expected invariants: circlehunter reaches F1 0.93 at >=30x local depth, outperforms Circle_finder at all depths except below 10x, and stays near-invariant to read length down to 35 bp, whereas Circle_finder fails below 75 bp reads.

## Related tools

- **circlehunter** (Primary ecDNA detector under evaluation; its Snakemake pipeline processes simulated ATAC-Seq libraries and its F1 scores are the benchmark output) — https://github.com/suda-huanglab/circlehunter
- **Circle_finder** (Comparator detector that collects junction reads from supplementary alignments; called on every simulated library to benchmark against circlehunter)
- **ART** (Simulator used to generate sequencing reads for 500 mock ecDNAs sampled from GRCh38)
- **Snakemake** (Workflow engine that runs the circlehunter pipeline on all simulated samples via the Snakefile and config.yaml)
- **ATAC-Seq** (Assay whose enhanced chromatin accessibility signal on ecDNA underpins detection and provides background reads for simulated libraries)

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- F1 at >=30x local depth reaches 0.93 for circlehunter on simulated data
- circlehunter F1 is near-invariant to read length down to 35 bp, while Circle_finder fails below 75 bp reads
- circlehunter outperforms Circle_finder at all tested local depths except below 10x
- All simulated samples and outputs are enumerated in config.yaml before launching Snakemake, ensuring complete condition coverage
- Per-condition precision and recall are computed against the exact simulated ground-truth ecDNA coordinates

## Limitations

- The current benchmarking strategy has limitations, and accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed
- Other classes of focal amplification with misregulated chromatin may be detected as false-positive ecDNA by circlehunter
- Short read-length sequencing may lead circlehunter to discover more computational sub-structures (e.g. COLO320DM)
- Simulated ecDNAs (5 kb-10 Mb from GRCh38 with 1-50 segments) may not capture all real ecDNA architectures

## Evidence

- [abstract] F1 invariance to read length: "circlehunter has an F1 score of 0.93 at 30× local depth and read lengths as short as 35 bp"
- [results] depth-sweep comparison against Circle_finder: [evidence span withheld — anchor status: not_found]
- [methods] pipeline launch command: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [readme] config file workflow: "circlehunter use a `yaml` format config file to figure out all samples and their output"
