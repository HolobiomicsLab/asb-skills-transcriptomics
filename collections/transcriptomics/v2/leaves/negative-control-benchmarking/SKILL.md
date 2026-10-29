---
name: negative-control-benchmarking
description: Use when assessing the false-positive rate of circlehunter ecDNA calls
  on non-cancer ATAC-Seq data (e.g., normal tissue) that lacks ecDNA amplification,
  or when validating a circlehunter installation before applying it to tumor samples.
when_to_use_negative:
- Input is tumor/cancer ATAC-Seq data expected to contain ecDNA amplification — this
  skill is a negative control for non-amplified samples only.
- Input is WGS data rather than ATAC-Seq — circlehunter exploits enhanced chromatin
  accessibility of ecDNA and is built for ATAC-Seq (WGS-based alternatives include
  AmpliconArchitect).
- Input is single-cell ATAC-Seq data — the application of circlehunter to single-cell
  ATAC-Seq data has yet to be explored.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: circlehunter
  role: The pipeline being benchmarked; run on normal-muscle ATAC-Seq data to quantify
    spurious ecDNA calls
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Workflow engine that executes the circlehunter pipeline per sample via the
    Snakefile and YAML config
  repo: https://snakemake.readthedocs.io/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:cf943c886d6d9b93e47dfa70dea71290e01a02b012c3d231e01154826f12bd8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# negative-control-benchmarking

## Summary

Reproduce the published negative-control benchmark for circlehunter by running its Snakemake ATAC-Seq pipeline on 14 normal-muscle-tissue ATAC-Seq datasets and confirming it reports essentially no ecDNA calls (only 1 call) on non-cancer data lacking ecDNA amplification.

## When to use

Use when assessing the false-positive rate of circlehunter ecDNA calls on non-cancer ATAC-Seq data (e.g., normal tissue) that lacks ecDNA amplification, or when validating a circlehunter installation before applying it to tumor samples.

## When NOT to use

- Input is tumor/cancer ATAC-Seq data expected to contain ecDNA amplification — this skill is a negative control for non-amplified samples only.
- Input is WGS data rather than ATAC-Seq — circlehunter exploits enhanced chromatin accessibility of ecDNA and is built for ATAC-Seq (WGS-based alternatives include AmpliconArchitect).
- Input is single-cell ATAC-Seq data — the application of circlehunter to single-cell ATAC-Seq data has yet to be explored.

## Inputs

- 14 normal-muscle-tissue ATAC-Seq datasets (public SRA accessions listed in Supplementary Table S1)
- environment.yml (conda/mamba dependency specification from the circlehunter repository)
- config-template.yaml (sample/output configuration template from the circlehunter repository)
- Snakefile (Snakemake workflow definition from the circlehunter repository)

## Outputs

- Per-sample circlehunter ecDNA call outputs for each of the 14 normal-muscle samples
- Summary table of ecDNA calls across all 14 samples (expected: exactly 1 ecDNA call)

## How to apply

Install all dependencies with mamba by creating the environment from environment.yml and activating the 'circlehunter' conda environment. Retrieve the 14 normal-muscle-tissue ATAC-Seq datasets corresponding to the accessions in Supplementary Table S1. Create a workspace directory, copy the provided config-template.yaml to config.yaml, and edit the YAML configuration to list all 14 normal-muscle samples and their output paths. Execute the pipeline with 'snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml', allowing up to 16 parallel processes. Collect the per-sample ecDNA calls from the pipeline outputs into a summary table and compare against the reported negative-control result of 1 ecDNA call across the 14 samples — rationale: ecDNA is typically found only in cancer samples, so a low call count on normal muscle tissue demonstrates the tool is unlikely to report non-amplified false positives.

## Related tools

- **circlehunter** (The pipeline being benchmarked; run on normal-muscle ATAC-Seq data to quantify spurious ecDNA calls) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Workflow engine that executes the circlehunter pipeline per sample via the Snakefile and YAML config) — https://snakemake.readthedocs.io/

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- The summary table must contain exactly 1 ecDNA call across the 14 normal-muscle samples, matching the reported negative-control result of few non-amplified false positives.
- All 14 normal-muscle samples listed in the config.yaml produced pipeline outputs with no missing samples.
- The mamba environment creation from environment.yml and 'conda activate circlehunter' completes without dependency resolution errors.
- The snakemake run with -j 16 finishes without rule failures, confirming the pipeline itself executed correctly on the control data.

## Limitations

- The current benchmarking strategy has limitations, and accuracy of ecDNA calls cannot be guaranteed for all types of focal amplifications.
- If other classes of focal amplification have misregulated chromatin, they may be detected as false-positive ecDNA by circlehunter.
- Short read-length sequencing may lead circlehunter to discover more computational sub-structures (e.g., COLO320DM).
- The negative control covers only normal-muscle tissue; broader normal-tissue coverage would strengthen the false-positive estimate.

## Evidence

- [readme] snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [methods] All dependencies can be installed using `mamba`: "All dependencies can be installed using `mamba`"
- [intro] ecDNA is typically found only in cancer samples, typically has a size over 1 million base pairs, and contains both genes and noncoding DNA: "ecDNA is typically found only in cancer samples, typically has a size over 1 million base pairs, and contains both genes and noncoding DNA"
- [discussion] if other classes of focal amplification have misregulated chromatin, they may be detected as false-positive ecDNA by circlehunter: "if other classes of focal amplification have misregulated chromatin, they may be detected as false-positive ecDNA by circlehunter"
- [abstract] We developed circlehunter ( https://github.com/suda-huanglab/circlehunter ), a tool for identifying ecDNA from ATAC-Seq data: "We developed circlehunter ( https://github.com/suda-huanglab/circlehunter ), a tool for identifying ecDNA from ATAC-Seq data"
