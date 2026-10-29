---
name: nextflow-workflow-execution
description: Use when a publication provides a Nextflow workflow repository (e.g.
  guigolab/sqtlseeker2-nf) and you need to reproduce its containerized analysis on
  your own per-tissue input files.
when_to_use_negative:
- The analysis is a single lightweight script or interactive notebook where workflow-engine
  overhead and containerization add no reproducibility value.
- The workflow repository requires Nextflow DSL1 but your environment only offers
  a recent Nextflow release without a pinned NXF_VER (DSL1 is no longer supported).
- The input data do not match the workflow's expected schema (e.g. transcript expression
  lacking trId/geneId columns or genotypes not sorted by coordinate).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: Nextflow
  role: Workflow engine that executes the sqtlseeker2-nf pipeline reproducibly; installed
    via get.nextflow.io and pinned with NXF_VER for DSL1 compatibility
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: sQTLseekeR2
  role: 'Underlying R tool invoked by the workflow steps: genotype indexing, transcript-expression
    preprocessing, nominal and permutation testing, multiple-testing correction'
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: The Nextflow pipeline repository being executed; launched with 'nextflow run
    guigolab/sqtlseeker2-nf -with-docker'
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: Docker
  role: Container runtime used via -with-docker (or Singularity) to pin the software
    environment of every pipeline step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:c0c6cd043828a35d083a4177e239e84377f210d82f07d8bfca1c16a50b2fdc5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sqtlseeker2-nf
  tools:
  - Docker
  - Nextflow
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# nextflow-workflow-execution

## Summary

Executes reproducible multi-step scientific pipelines (e.g. sQTL mapping with sqtlseeker2-nf) as Nextflow workflows run inside Docker or Singularity containers, ensuring identical tool versions and parameters across tissues or datasets.

## When to use

Use when a publication provides a Nextflow workflow repository (e.g. guigolab/sqtlseeker2-nf) and you need to reproduce its containerized analysis on your own per-tissue input files.

## When NOT to use

- The analysis is a single lightweight script or interactive notebook where workflow-engine overhead and containerization add no reproducibility value.
- The workflow repository requires Nextflow DSL1 but your environment only offers a recent Nextflow release without a pinned NXF_VER (DSL1 is no longer supported).
- The input data do not match the workflow's expected schema (e.g. transcript expression lacking trId/geneId columns or genotypes not sorted by coordinate).

## Inputs

- coordinate-sorted genotype file (0/1/2 coding, -1 for missing)
- transcript expression file with trId and geneId columns
- metadata file with indId, sampleId, group columns plus covariates
- gene location file with chr, start, end and geneId columns

## Outputs

- per-group result files under result/groups/<group>/ including all-tests.nominal.tsv
- per-group all-tests.permuted.tsv from the permutation pass
- significant sQTL file sqtls-0.05fdr.permuted.tsv with fdr column computed over empirical P-values

## How to apply

Install Nextflow pinned to a version that still supports DSL1 (e.g. NXF_VER=22.04.0, since recent releases dropped DSL1) via 'curl -fsSL get.nextflow.io | bash', then clone or pull the workflow repository (guigolab/sqtlseeker2-nf). Stage the required inputs — for sqtlseeker2-nf these are a coordinate-sorted genotype file (0/1/2 coding, -1 missing), a transcript expression file with trId and geneId columns, a metadata file with indId/sampleId/group columns plus covariates, and a gene location file with chr/start/end/geneId. Launch the pipeline with './nextflow run guigolab/sqtlseeker2-nf -with-docker' (or -with-singularity), passing --genotype, --trexp, --metadata, --genes and the output directory via --dir for each group defined in the metadata. Set analysis-mode parameters explicitly: --mode permuted with --max_perm 1000 for the adaptive permutation pass after the nominal pass, --covariates true for covariate regression, and thresholds such as --fdr 0.05 and --min_md 0.05. Rationale: containerization fixes tool versions so the per-group outputs under result/groups/<group>/ (all-tests.nominal.tsv, all-tests.permuted.tsv, sqtls-0.05fdr.permuted.tsv) are reproducible and comparable across tissues.

## Related tools

- **Nextflow** (Workflow engine that executes the sqtlseeker2-nf pipeline reproducibly; installed via get.nextflow.io and pinned with NXF_VER for DSL1 compatibility) — https://github.com/guigolab/sqtlseeker2-nf
- **sQTLseekeR2** (Underlying R tool invoked by the workflow steps: genotype indexing, transcript-expression preprocessing, nominal and permutation testing, multiple-testing correction) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (The Nextflow pipeline repository being executed; launched with 'nextflow run guigolab/sqtlseeker2-nf -with-docker') — https://github.com/guigolab/sqtlseeker2-nf
- **Docker** (Container runtime used via -with-docker (or Singularity) to pin the software environment of every pipeline step)

## Examples

```
NXF_VER=22.04.0 ./nextflow run guigolab/sqtlseeker2-nf -with-docker --genotype gt.vcf.gz --trexp tr.tsv --metadata md.tsv --genes genes.tsv --dir out --mode permuted --max_perm 1000 --covariates true --fdr 0.05
```

## Evaluation signals

- Each tissue's sqtls-0.05fdr.permuted.tsv exists and contains the fdr column computed across empirical P-values at the 0.05 threshold.
- Per-tissue output directories result/groups/<group>/ contain all three expected files: all-tests.nominal.tsv, all-tests.permuted.tsv and sqtls-0.05fdr.permuted.tsv.
- Aggregated per-tissue sQTL totals reproduce the published GTEx V7 catalog counts (210,485 sQTLs affecting 6,963 sGenes).
- sGene counts decompose into 6,685 protein-coding genes and 278 lincRNAs as reported.
- The pipeline log confirms both passes ran (nominal pass and permuted pass) with the requested parameters (--mode permuted, --max_perm 1000, --covariates true).

## Limitations

- Recent Nextflow releases no longer support DSL1, so the version must be pinned (e.g. NXF_VER=22.04.0) or the workflow will fail.
- The nominal and permuted passes process genes in batches (--kn and --kp, default 10 genes per batch); very small batches can slow genome-scale runs.
- LD-based variant clustering is off by default (--ld 0, no clustering) and must be enabled explicitly if LD grouping is desired.
- Per-tissue runs must be launched separately for each group defined in the metadata file; there is no single invocation across all tissues.

## Evidence

- [discussion] our implementation of an enhanced pipeline for sQTL mapping based on sQTLseekeR2, Nextflow and Docker: "our implementation of an enhanced pipeline for sQTL mapping based on sQTLseekeR2, Nextflow and Docker"
- [methods] nextflow run with docker: "./nextflow run guigolab/sqtlseeker2-nf -with-docker"
- [methods] minimum effect size parameter: "--min_md MIN_MD             minimum effect size reported (default: 0.05)"
- [readme] A pipeline for splicing quantitative trait loci mapping: "A pipeline for splicing quantitative trait loci (sQTL) mapping.

The pipeline performs the following analysis steps:

* Index the genotype file"
- [intro] workflow repository URL: "computational workflow named sqtlseeker2-nf, available at https://github.com/guigolab/sqtlseeker2-nf"
