---
name: snakemake-workflow-configuration
description: Use when you need to run the circlehunter ecDNA-calling pipeline on one
  or more ATAC-Seq samples (simulated, cell-line, or negative-control) via its Snakemake
  workflow and YAML configuration.
when_to_use_negative:
- The analysis goal is WGS-based ecDNA detection requiring copy-number input (e.g.,
  AmpliconArchitect, JaBbA, LINX), not ATAC-Seq-based ecDNA calling
- Input is single-cell ATAC-Seq data — the article notes its application to single-cell
  ATAC-Seq has yet to be explored
- You only need gene set enrichment analysis (GSEApy), which is a separate tool not
  part of this Snakemake ecDNA workflow
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: circlehunter
  role: The ecDNA-identification pipeline being orchestrated; its Snakefile and config-template.yaml
    define the workflow executed here
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Workflow engine used to run the circlehunter rules with the --snakefile, -j,
    and --configfile options
- name: mamba/conda
  role: Installs all pipeline dependencies from environment.yml into the circlehunter
    environment
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:79cc2011460895f5aa0407dc3b9209e092250c0f0b571c0a34c36199a2b288ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - ART
  - Circle_finder
  - Snakemake
  - circlehunter
  - mamba/conda
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# snakemake-workflow-configuration

## Summary

Configure and execute the circlehunter Snakemake pipeline, which identifies ecDNA from ATAC-Seq data, by installing the conda/mamba environment and driving the workflow from a YAML sample config file.

## When to use

Use when you need to run the circlehunter ecDNA-calling pipeline on one or more ATAC-Seq samples (simulated, cell-line, or negative-control) via its Snakemake workflow and YAML configuration.

## When NOT to use

- The analysis goal is WGS-based ecDNA detection requiring copy-number input (e.g., AmpliconArchitect, JaBbA, LINX), not ATAC-Seq-based ecDNA calling
- Input is single-cell ATAC-Seq data — the article notes its application to single-cell ATAC-Seq has yet to be explored
- You only need gene set enrichment analysis (GSEApy), which is a separate tool not part of this Snakemake ecDNA workflow

## Inputs

- ATAC-Seq sequencing samples (e.g., GEO accessions such as ENCODE GSE170245 GM12878 background reads, GBM39/PC3/COLO320DM ATAC-Seq)
- circlehunter config-template.yaml (YAML sample/output configuration template)
- environment.yml (conda/mamba dependency specification)
- Snakefile from the circlehunter repository

## Outputs

- Per-sample ecDNA calls with reconstructed circular structures, ligation relationships, and breakpoints
- Pipeline outputs enumerated per sample in the YAML config (e.g., ecDNA call tables for comparison against ground truth or Fig. 2D-F structures)

## How to apply

First install all dependencies by creating the environment from environment.yml with mamba (`mamba env create -f environment.yml`) and activate it with `conda activate circlehunter`. Then create an analysis workspace (`mkdir workspace; cd workspace`) and copy the provided template with `cp ../config-template.yaml config.yaml`. Edit the YAML config to enumerate all samples — e.g., three GEO ATAC-Seq samples (GBM39, PC3, COLO320DM) or 14 normal-muscle samples — together with their desired output paths. Launch the workflow with `snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml`, where `-j 16` sets the maximum number of parallel processes and `--configfile` points to your edited YAML. Correct application yields per-sample ecDNA calls whose structures, ligation relationships, and breakpoints can be compared against known references (e.g., WGS/FISH-validated ecDNA in Fig. 2D-F).

## Related tools

- **circlehunter** (The ecDNA-identification pipeline being orchestrated; its Snakefile and config-template.yaml define the workflow executed here) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Workflow engine used to run the circlehunter rules with the --snakefile, -j, and --configfile options)
- **mamba/conda** (Installs all pipeline dependencies from environment.yml into the circlehunter environment)

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- The conda environment 'circlehunter' activates successfully after `mamba env create -f environment.yml`
- Snakemake completes without errors using `snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml` for all samples listed in the YAML
- On the three validated cell lines (GBM39, PC3, COLO320DM), reconstructed structures reproduce the WGS/FISH-validated ecDNA in Fig. 2D-F, including all ligation relationships for PC3's six-fragment ecDNA
- On the 14 normal-muscle negative-control ATAC-Seq datasets, the summary table contains exactly 1 ecDNA call (few false positives)
- Per-sample outputs match the paths enumerated in the edited config.yaml

## Limitations

- Accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed; the benchmarking strategy has limitations, and other classes of focal amplification with misregulated chromatin may be detected as false-positive ecDNA
- Short read-length sequencing may lead circlehunter to discover more computational sub-structures (e.g., COLO320DM)
- The -j 16 flag caps parallel processes; check the Snakemake documentation for other execution options

## Evidence

- [readme] circlehunter use a `yaml` format config file to figure out all samples and their output: "circlehunter use a `yaml` format config file to figure out all samples and their output, to run the analysis processes, a config file is needed."
- [methods] snakemake command with configfile: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [methods] environment installation via mamba: "All dependencies can be installed using `mamba`"
- [readme] workspace setup: "# make a directory for analysis
mkdir workspace
cd workspace
# copy and edit the config file
cp ../config-template.yaml config.yaml"
