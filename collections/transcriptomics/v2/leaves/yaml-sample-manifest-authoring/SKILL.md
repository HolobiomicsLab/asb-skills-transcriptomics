---
name: yaml-sample-manifest-authoring
description: Use when running the circlehunter ATAC-Seq ecDNA pipeline on a new cohort
  (e.g., 14 normal-muscle negative controls or 547 GEO tumor samples) and a sample
  manifest must list each sample and its output path.
when_to_use_negative:
- You need to change pipeline logic or rule dependencies rather than the sample cohort
  — edit the Snakefile, not the YAML manifest.
- You are running GSEApy or another tool whose configuration is not the circlehunter
  YAML format; the circlehunter config schema is specific to its Snakefile.
- Only a single pre-configured sample is being rerun unchanged and a valid config.yaml
  already exists.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: circlehunter
  role: Pipeline whose YAML config schema defines the manifest format; each entry
    names one ATAC-Seq sample and its output path
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Workflow engine that consumes the authored config.yaml via --configfile to
    run the pipeline on all listed samples
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:55a7d7429ba25fe3c58709ab9b36586f9e735990bc3a66163dcadb55dad5c5a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# yaml-sample-manifest-authoring

## Summary

Authoring the YAML sample manifest (config.yaml) that drives the circlehunter Snakemake pipeline: copying the provided config-template.yaml into a workspace and editing it to enumerate every ATAC-Seq sample and its output path so the pipeline knows which datasets to process.

## When to use

Use when running the circlehunter ATAC-Seq ecDNA pipeline on a new cohort (e.g., 14 normal-muscle negative controls or 547 GEO tumor samples) and a sample manifest must list each sample and its output path.

## When NOT to use

- You need to change pipeline logic or rule dependencies rather than the sample cohort — edit the Snakefile, not the YAML manifest.
- You are running GSEApy or another tool whose configuration is not the circlehunter YAML format; the circlehunter config schema is specific to its Snakefile.
- Only a single pre-configured sample is being rerun unchanged and a valid config.yaml already exists.

## Inputs

- circlehunter config-template.yaml (template from suda-huanglab/circlehunter repo)
- Sample list (e.g., GEO/SRA ATAC-Seq accessions from Supplementary Table S1 or curated tumor dataset)
- Desired per-sample output paths

## Outputs

- config.yaml (YAML-format sample manifest enumerating all samples and their output paths, consumed by the Snakemake Snakefile)

## How to apply

Install dependencies with mamba from the circlehunter environment.yml and activate the environment first. Create a workspace directory, copy the repository's config-template.yaml to config.yaml ('cp ../config-template.yaml config.yaml'), then edit the YAML to 'figure out all samples and their output' — i.e., list every sample (e.g., the 14 normal-muscle ATAC-Seq accessions from Supplementary Table S1, or all curated GEO tumor samples) together with its output path. Keep the YAML structure defined by the template so Snakemake can parse it with --configfile. The manifest is the single decision point controlling cohort composition: once samples are declared, launch the pipeline with 'snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml' and the -j 16 option caps parallel processes at 16.

## Related tools

- **circlehunter** (Pipeline whose YAML config schema defines the manifest format; each entry names one ATAC-Seq sample and its output path) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Workflow engine that consumes the authored config.yaml via --configfile to run the pipeline on all listed samples)

## Examples

```
mkdir workspace; cd workspace; cp ../config-template.yaml config.yaml; snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- config.yaml parses as valid YAML and preserves the key structure of config-template.yaml.
- The manifest enumerates the full intended cohort (e.g., all 14 normal-muscle samples, or all 547 curated tumor samples) with one output path per sample and no missing/duplicate entries.
- The pipeline launches without config errors and produces per-sample output directories for every manifest entry.
- Downstream validation succeeds: negative-control run yields exactly 1 ecDNA call across 14 normal-muscle samples; tumor cohort run yields 1312 candidate ecDNAs in 94 (17.18%) samples of 17 cancers.

## Limitations

- The manifest only controls which samples run; it does not adjust detection sensitivity — circlehunter may call false-positive ecDNA for other focal amplification classes with misregulated chromatin.
- circlehunter cannot identify ecDNA without the increased ATAC-Seq signal of high chromatin accessibility, so samples lacking such signal will be uninformative regardless of manifest content.
- Parallelism is capped by the -j option (16 in the documented runs); a very large manifest will still run at most 16 processes at a time.

## Evidence

- [methods] circlehunter use a `yaml` format config file to figure out all samples and their output: "circlehunter use a `yaml` format config file to figure out all samples and their output"
- [methods] copy config template into workspace: "mkdir workspace
cd workspace
cp ../config-template.yaml config.yaml"
- [methods] run command with configfile and parallelism: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [readme] README: edit config to fit requirements: "modify the configuration file according to the template to fit your requirements, then run the pipeline by"
