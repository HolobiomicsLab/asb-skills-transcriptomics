---
name: containerized-pipeline-execution
description: Use when executing the SkewC workflow (0_split10XbyBarcode.sh, 1_geneBodyCoverage.sh,
  2_SkewC.sh) on pbmc_8k or other 10x/ E-MTAB-2600 data, especially on systems without
  admin rights where the R 'stringi'/libicui18n.so.
when_to_use_negative:
- You already have a compatible local R/Perl environment that runs the SkewC bash
  scripts without errors — containerization adds overhead without benefit.
- Your task is only user-space image handling (e.g. udocker search/pull/export) unrelated
  to running the SkewC pipeline.
- BAM files are already split by cells and you only need the R Markdown analysis —
  the containerized split step (0_split10XbyBarcode.sh) can be skipped per the README.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: docker
  role: Primary container runtime for executing the SkewC bash pipeline steps; recommended
    when the user has admin authority on the machine.
- name: udocker
  role: Runs the SkewC docker container in user space without root privileges on Linux
    batch/cluster systems where docker is unavailable.
  repo: https://github.com/indigo-dc/udocker
- name: singularity
  role: Recommended runtime; build a Singularity Image File (SIF) from the docker://moirai2/skewc
    image before running the pipeline.
- name: SkewC
  role: The R/Perl scRNA-seq quality-control workflow that is containerized and executed
    (split BAM, geneBodyCoverage.pl, trim clustering).
  repo: https://github.com/LSBDT/SkewC.git
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.xpro.2022.102038
    title: Computational approach to evaluate scRNA-seq data quality and gene body
      coverage with SkewC
  source_content_hash: sha256:ddedd159a6aa2ee01e424233dea27e2c96523b4081eba7d53f7712738e822f13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/indigo-dc/udocker
  tools:
  - R
  - SkewC
  - docker
  - singularity
  - udocker
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# containerized-pipeline-execution

## Summary

Run the SkewC scRNA-seq quality-control pipeline (splitting 10x barcoded BAMs, geneBodyCoverage.pl, and R Markdown clustering steps) inside a Docker, udocker, or Singularity container so that the R/Perl stack and dependencies are reproducible. The article ships the workflow as a docker/udocker container and a Singularity Image File (SIF) precisely because the R and Perl implementations are fragile to local installation differences.

## When to use

Use when executing the SkewC workflow (0_split10XbyBarcode.sh, 1_geneBodyCoverage.sh, 2_SkewC.sh) on pbmc_8k or other 10x/ E-MTAB-2600 data, especially on systems without admin rights where the R 'stringi'/libicui18n.so.57 error or missing git/docker/udocker/singularity would otherwise block a local install.

## When NOT to use

- You already have a compatible local R/Perl environment that runs the SkewC bash scripts without errors — containerization adds overhead without benefit.
- Your task is only user-space image handling (e.g. udocker search/pull/export) unrelated to running the SkewC pipeline.
- BAM files are already split by cells and you only need the R Markdown analysis — the containerized split step (0_split10XbyBarcode.sh) can be skipped per the README.

## Inputs

- 10x Genomics barcoded BAM file (e.g. pbmc_8k barcoded.bam) plus barcodes.tsv.gz from outs/filtered_feature_bc_matrix/
- Reference BED file (reference/hg38_Gencode_V28.norRNAtRNA.bed or mm10_Gencode_VM18.norRNAtRNA.bed)
- Container runtime environment: docker, udocker, or singularity with the moirai2/skewc image

## Outputs

- Per-cell BAM files under the split output directory (default 'input')
- Gene body coverage files from geneBodyCoverage.pl under 'coverage'
- Final SkewC HTML report under skewc/

## How to apply

First clone the SkewC repository (git clone https://github.com/LSBDT/SkewC.git), which requires git to be installed — cloning fails when git is missing. Then choose a container runtime: docker if you have admin authority, udocker if you run on Linux without admin rights, or singularity (recommended by the README), which needs an extra conversion step: singularity build skewc.sif docker://moirai2/skewc:latest. Execute the bash steps (0_split10XbyBarcode.sh with $bam, $barcode, $outdir; 1_geneBodyCoverage.sh with species hg38/mm10; 2_SkewC.sh) from the SkewC root via container_bash.sh so the R and Perl scripts (geneBodyCoverage.pl, split10XbyBarcode.pl, SkewC.r) run against the pinned container environment. Rationale: the paper reports the workflow 'is provided as a docker (and udocker) container and as a Singularity Image File (SIF)' to avoid user-side installation problems, including the R package error where SkewC terminates and is 'not fixable from the user's side'.

## Related tools

- **docker** (Primary container runtime for executing the SkewC bash pipeline steps; recommended when the user has admin authority on the machine.)
- **udocker** (Runs the SkewC docker container in user space without root privileges on Linux batch/cluster systems where docker is unavailable.) — https://github.com/indigo-dc/udocker
- **singularity** (Recommended runtime; build a Singularity Image File (SIF) from the docker://moirai2/skewc image before running the pipeline.)
- **SkewC** (The R/Perl scRNA-seq quality-control workflow that is containerized and executed (split BAM, geneBodyCoverage.pl, trim clustering).) — https://github.com/LSBDT/SkewC.git

## Examples

```
singularity build skewc.sif docker://moirai2/skewc:latest
```

## Evaluation signals

- The image build succeeds: skewc.sif is produced from docker://moirai2/skewc:latest without error.
- bash 0_split10XbyBarcode.sh $bam $barcode $outdir runs without error and the number of per-cell BAM files equals the number of entries in barcodes.tsv.
- 1_geneBodyCoverage.sh generates an index file under reference/ on first run (hg38 13MB bed file => 82MB index file; mm10 9MB => 60MB) and coverage outputs under the default 'coverage' directory.
- Final HTML output appears under skewc/ after 2_SkewC.sh, indicating the containerized run completed.
- No R 'stringi/libicui18n.so.57' termination error occurs, confirming the container's R environment is intact.

## Limitations

- The R package error (stringi/libicui18n.so.57) terminates SkewC and is not fixable from the user's side — the container is the mitigation, but a broken image will still fail.
- SkewC requires git and at least one of docker, udocker, or singularity to be installed; installation/cloning fails when these are missing.
- Singularity requires the extra conversion step to build an SIF from the docker image, and udocker/singularity may not be present on the user's system at all.

## Evidence

- [abstract] workflow containerization: "the workflow is provided as a docker (and udocker) container and as a Singularity Image File (SIF)"
- [methods] singularity build: "singularity build skewc.sif docker://moirai2/skewc:latest"
- [discussion] prerequisite failure: [evidence span withheld — anchor status: not_found]
- [readme] repo clone: "git clone https://github.com/LSBDT/SkewC.git"
- [readme] udocker use case: "Install udocker, if you want to run pipeline in Linux environment where you don't have any admin authority (and can't run docker)."
