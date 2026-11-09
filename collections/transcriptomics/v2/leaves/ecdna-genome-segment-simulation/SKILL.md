---
name: ecdna-genome-segment-simulation
description: Use when benchmarking ecDNA-calling tools on ATAC-Seq data and you need
  simulated ground-truth ecDNA amplicons spanning controlled local depths (e.g. 10×–30×+)
  and read lengths (e.g. 35–150 bp).
when_to_use_negative:
- Evaluating ecDNA calls from real tumor samples where no ground-truth ecDNA coordinates
  exist — use orthogonal validation instead
- Benchmarking whole-genome-sequencing-based ecDNA detectors such as AmpliconArchitect,
  which require copy-number input rather than ATAC-Seq reads
- Detecting ecDNA in a single analysis sample rather than quantifying detector accuracy
  across depth/read-length sweeps
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: ART
  role: Simulates 500 mock ecDNAs from GRCh38 (5 kb–10 Mb, 1–50 segments) and generates
    their sequencing reads blended with GM12878 ATAC-Seq background
- name: circlehunter
  role: Detector under benchmarking; identifies ecDNA from the simulated ATAC-Seq
    libraries for precision/recall/F1 comparison
  repo: https://github.com/suda-huanglab/circlehunter
- name: Circle_finder
  role: Comparator detector run on every simulated library to benchmark circlehunter
    against
- name: Snakemake
  role: Executes the pipeline across all simulated samples via the Snakefile and YAML
    config
- name: ATAC-Seq
  role: Assay whose enhanced chromatin accessibility on ecDNA provides the signal
    the simulated libraries emulate
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:906130f8bda1ca6136ba6f66270915fc893bcedf4a1333f331714be00e8aada5
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# ecdna-genome-segment-simulation

## Summary

Simulate mock ecDNAs from GRCh38 with ART to generate ground-truth ATAC-Seq libraries for benchmarking ecDNA detectors such as circlehunter against Circle_finder across local depth and read-length conditions.

## When to use

Use when benchmarking ecDNA-calling tools on ATAC-Seq data and you need simulated ground-truth ecDNA amplicons spanning controlled local depths (e.g. 10×–30×+) and read lengths (e.g. 35–150 bp).

## When NOT to use

- Evaluating ecDNA calls from real tumor samples where no ground-truth ecDNA coordinates exist — use orthogonal validation instead
- Benchmarking whole-genome-sequencing-based ecDNA detectors such as AmpliconArchitect, which require copy-number input rather than ATAC-Seq reads
- Detecting ecDNA in a single analysis sample rather than quantifying detector accuracy across depth/read-length sweeps

## Inputs

- GRCh38 reference genome (source of simulated ecDNA segments)
- GM12878 ATAC-Seq background reads (ENCODE GSE170245)
- YAML config file enumerating simulated samples and outputs (copied from config-template.yaml)
- Snakemake Snakefile (../Snakefile)

## Outputs

- Simulated ecDNA FASTQ read libraries at controlled local depths and read lengths
- Ground-truth simulated ecDNA coordinate sets
- Per-tool ecDNA calls from circlehunter and Circle_finder on each simulated library
- F1 score table per depth and read-length condition (f1_scores_fig2ab.csv)

## How to apply

Install the circlehunter environment from environment.yml with mamba (mamba env create -f environment.yml; conda activate circlehunter). Use ART to simulate 500 mock ecDNAs sampled from GRCh38, each 5 kb–10 Mb in size and composed of 1–50 genomic segments, and generate their sequencing reads. Blend the simulated ecDNA reads with GM12878 ATAC-Seq background reads (ENCODE GSE170245) to build test libraries spanning varying local depths and read lengths. Configure a YAML config enumerating all simulated samples and outputs, run the Snakemake pipeline, then call ecDNA with both circlehunter and Circle_finder on every library. Compare each detector's calls against the simulated ground-truth ecDNA coordinates and compute precision, recall, and F1 per depth and read-length condition, assembling the results into f1_scores_fig2ab.csv.

## Related tools

- **ART** (Simulates 500 mock ecDNAs from GRCh38 (5 kb–10 Mb, 1–50 segments) and generates their sequencing reads blended with GM12878 ATAC-Seq background)
- **circlehunter** (Detector under benchmarking; identifies ecDNA from the simulated ATAC-Seq libraries for precision/recall/F1 comparison) — https://github.com/suda-huanglab/circlehunter
- **Circle_finder** (Comparator detector run on every simulated library to benchmark circlehunter against)
- **Snakemake** (Executes the pipeline across all simulated samples via the Snakefile and YAML config)
- **ATAC-Seq** (Assay whose enhanced chromatin accessibility on ecDNA provides the signal the simulated libraries emulate)

## Examples

```
mamba env create -f environment.yml; conda activate circlehunter; cp ../config-template.yaml config.yaml; snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- circlehunter reaches an F1 score of 0.93 at local depth greater than 30× on the simulated libraries
- F1 is near-invariant to read lengths down to 35 bp for circlehunter, while Circle_finder fails below 75 bp reads
- Precision, recall, and F1 computed against ground-truth simulated ecDNA coordinates per depth and read-length condition are assembled in f1_scores_fig2ab.csv
- circlehunter outperforms Circle_finder at all local depth levels except below 10×
- All 500 simulated ecDNAs fall within 5 kb–10 Mb and are composed of 1–50 genomic segments

## Limitations

- The current benchmarking strategy has limitations and accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed
- Simulated ecDNAs rely on increased ATAC-Seq accessibility; other focal amplification classes with misregulated chromatin may be detected as false positives
- Short read-length sequencing may lead circlehunter to discover computational sub-structures (e.g. COLO320DM)

## Evidence

- [abstract] circlehunter has an F1 score of 0.93 at 30× local depth and read lengths as short as 35 bp: "circlehunter has an F1 score of 0.93 at 30× local depth and read lengths as short as 35 bp"
- [methods] snakemake pipeline invocation: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [methods] config template setup: "cp ../config-template.yaml config.yaml"
- [discussion] benchmarking limitation: "it is important to note that the current benchmarking strategy has limitations, and we cannot guarantee the accuracy of ecDNA calls for all types of focal amplifications"
- [readme] mamba environment install from README: "All dependencies can be installed using `mamba`"
