---
name: conda-environment-reproducibility
description: Use when reproducing the circlehunter ecDNA-detection workflow or any
  Snakemake pipeline whose dependencies are enumerated in an environment.yml and must
  be installed identically across machines.
when_to_use_negative:
- The pipeline dependencies are already present and pinned in an active environment,
  so re-creating from environment.yml would duplicate work
- The target tool is not distributed with an environment.yml or equivalent dependency
  lockfile — ad hoc conda/pip installs would be required instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0769
tools:
- name: mamba
  role: Package manager used to create the reproducible 'circlehunter' conda environment
    from environment.yml
- name: Snakemake
  role: Workflow engine executed inside the activated environment to run the circlehunter
    rules with config.yaml
- name: circlehunter
  role: The pipeline whose dependency set is captured in environment.yml and executed
    via Snakemake to identify ecDNA from ATAC-Seq data
  repo: https://github.com/suda-huanglab/circlehunter
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:e3f96a90ad2f7fd62405dff8d4d29480b8bf2365aca194a896e92f5c2a561441
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - Snakemake
  - circlehunter
  - mamba
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# conda-environment-reproducibility

## Summary

Builds an isolated, reproducible software environment from a declarative environment.yml file using mamba/conda so that the circlehunter ATAC-Seq ecDNA detection pipeline (Snakemake-based) runs with pinned dependencies.

## When to use

Use when reproducing the circlehunter ecDNA-detection workflow or any Snakemake pipeline whose dependencies are enumerated in an environment.yml and must be installed identically across machines.

## When NOT to use

- The pipeline dependencies are already present and pinned in an active environment, so re-creating from environment.yml would duplicate work
- The target tool is not distributed with an environment.yml or equivalent dependency lockfile — ad hoc conda/pip installs would be required instead

## Inputs

- environment.yml conda dependency specification from the suda-huanglab/circlehunter repository
- config-template.yaml pipeline configuration template (copied to config.yaml and edited to list ATAC-Seq samples and desired outputs)
- Snakefile workflow definition (referenced as ../Snakefile relative to the workspace)

## Outputs

- A named conda environment 'circlehunter' containing all pipeline dependencies (including Snakemake and mamba-installed tools)
- Snakemake-declared pipeline outputs for every sample entry in config.yaml

## How to apply

Install mamba into the conda base environment from the conda-forge channel (`conda install -n base -c conda-forge mamba`), then create the dedicated environment from the repository's environment.yml with `mamba env create -f environment.yml` and activate it via `conda activate circlehunter`. This pulls every dependency of the circlehunter pipeline (including Snakemake) into one named, version-managed environment, isolating it from the base installation. Once active, the analysis workspace is prepared by copying the provided config-template.yaml to config.yaml and editing it to enumerate samples, after which the workflow is executed with `snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml`, where -j sets the maximum number of parallel processes. Creating the environment before any analysis is the rationale for reproducibility: all tools and versions are resolved once from environment.yml rather than ad hoc installs.

## Related tools

- **mamba** (Package manager used to create the reproducible 'circlehunter' conda environment from environment.yml)
- **Snakemake** (Workflow engine executed inside the activated environment to run the circlehunter rules with config.yaml)
- **circlehunter** (The pipeline whose dependency set is captured in environment.yml and executed via Snakemake to identify ecDNA from ATAC-Seq data) — https://github.com/suda-huanglab/circlehunter

## Examples

```
conda install -n base -c conda-forge mamba; mamba env create -f environment.yml; conda activate circlehunter
```

## Evaluation signals

- `conda activate circlehunter` succeeds, confirming the environment was created from environment.yml without solver errors
- `snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml` runs to completion with all rules executing
- The pipeline has produced its declared outputs for every sample entry in config.yaml
- A rerun on the same inputs reproduces the same ecDNA reconstructions (e.g., circlehunter accurately reproduced the WGS/FISH-validated GBM39, PC3, and COLO320DM ecDNA structures)

## Limitations

- circlehunter's current benchmarking strategy has limitations and accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed
- Application of circlehunter to single-cell ATAC-Seq data has yet to be explored
- Short read-length sequencing may lead circlehunter to discover more computational sub-structures (e.g., COLO320DM)

## Evidence

- [methods] mamba env create -f environment.yml: "All dependencies can be installed using `mamba`"
- [methods] snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [readme] cp ../config-template.yaml config.yaml: "circlehunter use a `yaml` format config file to figure out all samples and their output, to run the analysis processes, a config file is needed."
- [readme] conda activate circlehunter: "# create new environment according to environment.yml
mamba env create -f environment.yml
# active the new environment
conda activate circlehunter"
- [other] validation criterion: all Snakemake rules complete and declared outputs produced: [evidence span withheld — anchor status: not_found]
