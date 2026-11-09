---
name: atac-seq-ecdna-detection
description: Use when bulk ATAC-Seq datasets (tumor samples, cell lines, or normal
  negative controls) need screening for ecDNA/amplified circular structures and oncogene
  content, especially when WGS data or copy-number inputs are unavailable.
when_to_use_negative:
- Input is whole-genome sequencing rather than ATAC-Seq — the article contrasts circlehunter
  with WGS-based frameworks like AmpliconArchitect, which require copy number variation
  as input
- Input is single-cell ATAC-Seq data — the application of circlehunter to single-cell
  ATAC-Seq has yet to be explored
- The target is non-ecDNA classes of focal amplification or complex structural variants
  — accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed,
  and other focal amplification classes with misregulated chromatin may be reported
  as false-positive ecDNA
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3391
tools:
- name: circlehunter
  role: Core pipeline that identifies ecDNA from ATAC-Seq data by leveraging the increased
    ATAC-Seq signals from ecDNA's high chromatin accessibility
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Workflow engine executing the Snakefile, with the -j 16 option capping the
    maximum number of parallel processes
  repo: https://snakemake.readthedocs.io/
- name: mamba
  role: Installs all pipeline dependencies from environment.yml into the dedicated
    'circlehunter' conda environment
- name: AmpliconArchitect
  role: WGS-based ecDNA identification framework cited as the alternative approach
    requiring copy-number input, used for comparison and validation context
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_007
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:0dd94397b7df380634a54cca7f7a8b06b13504d537717e1a5e776ca25aa1e189
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - AmpliconArchitect
  - Snakemake
  - circlehunter
  - mamba
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# atac-seq-ecdna-detection

## Summary

A reproducible Snakemake-based pipeline (circlehunter) that reconstructs extrachromosomal circular DNA (ecDNA) from ATAC-Seq data by exploiting the elevated chromatin accessibility of ecDNA, avoiding RCA enrichment and resource-intensive WGS. It takes yaml-configured ATAC-Seq samples and produces per-sample ecDNA calls with segments, ligation relationships, and breakpoints.

## When to use

Use when bulk ATAC-Seq datasets (tumor samples, cell lines, or normal negative controls) need screening for ecDNA/amplified circular structures and oncogene content, especially when WGS data or copy-number inputs are unavailable.

## When NOT to use

- Input is whole-genome sequencing rather than ATAC-Seq — the article contrasts circlehunter with WGS-based frameworks like AmpliconArchitect, which require copy number variation as input
- Input is single-cell ATAC-Seq data — the application of circlehunter to single-cell ATAC-Seq has yet to be explored
- The target is non-ecDNA classes of focal amplification or complex structural variants — accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed, and other focal amplification classes with misregulated chromatin may be reported as false-positive ecDNA

## Inputs

- Public (GEO/SRA) bulk ATAC-Seq FASTQ datasets, e.g. 547 curated tumor samples or 14 normal-muscle negative-control samples listed in Supplementary Table S1
- config.yaml edited from config-template.yaml listing all samples and their desired outputs
- environment.yml dependency specification
- Snakemake Snakefile from the circlehunter repository

## Outputs

- Per-sample ecDNA calls: reconstructed circular structures with fragments/segments, ligation relationships, and predicted breakpoints
- Cohort-level summary table (e.g. total ecDNA count, number and percentage of ecDNA-positive samples, size/chromosome/segment-count distributions), e.g. 1312 calls in 94 of 547 samples (17.18%) across 17 cancers
- Oncogene content of predicted ecDNA (e.g. 37 oncogenes with amplification characteristics, including ecMYC/ecMYCN calls in SCLC cell lines)

## How to apply

Install all dependencies with mamba into a dedicated conda environment ('conda install -n base -c conda-forge mamba; mamba env create -f environment.yml; conda activate circlehunter'). Create a workspace directory, copy config-template.yaml to config.yaml, and edit the yaml config to enumerate all samples and their output paths — circlehunter uses this yaml file to figure out all samples and their outputs. Run the pipeline with 'snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml', where -j caps parallel processes at 16. Aggregate per-sample ecDNA calls into cohort statistics (total call count, number/percentage of ecDNA-positive samples, size, chromosome, and segment-count distributions) and inspect circular structures, ligation relationships, and breakpoints against WGS/FISH-validated references (e.g. GBM39, PC3, COLO320DM in Fig. 2D-F). Benchmark expectation: F1 = 0.93 at 30× local depth with reads as short as 35 bp on simulated data.

## Related tools

- **circlehunter** (Core pipeline that identifies ecDNA from ATAC-Seq data by leveraging the increased ATAC-Seq signals from ecDNA's high chromatin accessibility) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Workflow engine executing the Snakefile, with the -j 16 option capping the maximum number of parallel processes) — https://snakemake.readthedocs.io/
- **mamba** (Installs all pipeline dependencies from environment.yml into the dedicated 'circlehunter' conda environment)
- **AmpliconArchitect** (WGS-based ecDNA identification framework cited as the alternative approach requiring copy-number input, used for comparison and validation context)

## Examples

```
mamba env create -f environment.yml; conda activate circlehunter; snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- Negative-control invariant: applied to 14 normal-muscle-tissue ATAC-Seq datasets, the summary table contains exactly 1 ecDNA call, matching the reported negative-control result
- Cohort invariant: the 547-sample tumor run yields 1312 candidate ecDNA (or microDNA) circular structures in 94 (17.18%) samples across 17 cancers (Supplementary Table S3)
- Benchmark accuracy: F1 = 0.93 at 30× local depth with read lengths as short as 35 bp on simulated data
- Structure recovery: reconstructed circular structures, ligation relationships, and breakpoints match the WGS/FISH-validated ecDNA in Fig. 2D-F (simple GBM39 ecDNA; all ligation relationships and most breakpoints of PC3's six-fragment ecDNA)
- Execution completeness: all Snakemake rules complete and the pipeline produces its declared outputs for every sample entry in config.yaml

## Limitations

- The current benchmarking strategy has limitations, and accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed
- Short read-length sequencing disadvantages may cause circlehunter to discover more computational sub-structures (e.g. COLO320DM)
- Other classes of focal amplification with misregulated chromatin may be detected as false-positive ecDNA
- ecDNA lacking sufficient junction/structural information may not be identified by circlehunter
- Most analyzed samples were cell lines and cancer-type proportions deviated significantly; application to single-cell ATAC-Seq is unexplored

## Evidence

- [abstract] F1 benchmark on simulated data: "circlehunter has an F1 score of 0.93 at 30× local depth and read lengths as short as 35 bp"
- [intro] Rationale for ATAC-Seq-based ecDNA reconstruction: "we developed circlehunter to reconstruct ecDNA by taking full advantage of the increased ATAC-Seq signals arising from the high accessibility of ecDNA itself"
- [methods] Pipeline execution command: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [readme] Yaml config drives sample enumeration: "circlehunter use a `yaml` format config file to figure out all samples and their output"
- [readme] Workspace and config setup: "mkdir workspace
cd workspace
cp ../config-template.yaml config.yaml"
- [discussion] Short-read false-substructure caveat: "short read-length sequencing has a natural disadvantage for detecting structural variants, which may lead circlehunter to discover more computational sub-structures"
