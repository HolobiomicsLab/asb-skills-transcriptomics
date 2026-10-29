---
name: sqtl-mapping-pipeline-execution
description: Use when you need to identify genetic variants associated with multivariate
  splicing phenotypes (intron excision ratios) from genotype and transcript expression
  data in a reproducible, containerized workflow.
when_to_use_negative:
- The phenotype of interest is gene expression level rather than splicing ratios —
  an eQTL pipeline, not this sQTL mapper, is the right tool
- You need to splice individual splicing events univariately or perform co-localization/fine-mapping,
  which are not directly applicable within this multivariate, non-parametric framework
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: sQTLseekeR2
  role: Core R engine performing genotype indexing, expression preprocessing, multivariate
    cis-association testing, covariate correction, and permutation-based multiple
    testing correction
  repo: https://github.com/guigolab/sQTLseekeR2
- name: Nextflow
  role: Workflow engine orchestrating the pipeline steps reproducibly (sqtlseeker2-nf)
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: Docker
  role: Container runtime used via -with-docker to pin the pipeline software environment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:139fca649fa0993f68175e8766406c97a4304363f681360620fff73dded97411
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - Docker
  - Nextflow
  - sQTLseekeR2
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sqtl-mapping-pipeline-execution

## Summary

Execute the sQTLseekeR2-based Nextflow pipeline (sqtlseeker2-nf) to map splicing QTLs from genotype and transcript expression data, running a nominal cis-association pass followed by an optional permutation pass for empirical P-values and multiple-testing correction.

## When to use

Use when you need to identify genetic variants associated with multivariate splicing phenotypes (intron excision ratios) from genotype and transcript expression data in a reproducible, containerized workflow.

## When NOT to use

- The phenotype of interest is gene expression level rather than splicing ratios — an eQTL pipeline, not this sQTL mapper, is the right tool
- You need to splice individual splicing events univariately or perform co-localization/fine-mapping, which are not directly applicable within this multivariate, non-parametric framework

## Inputs

- Genotype file with 0/1/2/-1 genotype coding sorted by coordinate
- Transcript expression file with trId and geneId columns (e.g. from RSEM quantification)
- Metadata file with indId/sampleId/group columns
- Gene location file with chr/start/end/geneId columns

## Outputs

- Nominal-pass sQTL results with associations between variants and splicing ratios in cis
- Empirical P-values per phenotype from the permutation pass (beta-distribution fitted)
- Multiple-testing-corrected sQTL calls at the chosen FDR threshold (default 0.05)

## How to apply

Prepare four inputs: a genotype file with 0/1/2/-1 genotype coding sorted by coordinate, a transcript expression file with trId and geneId columns, a metadata file with indId/sampleId/group columns, and a gene location file with chr/start/end/geneId columns. Launch the pipeline via Nextflow inside Docker (e.g. NXF_VER=22.04.0 ./nextflow run guigolab/sqtlseeker2-nf -with-docker), which indexes the genotype file, preprocesses transcript expression, tests variants in cis (nominal pass, default win=5000 bp, fdr=0.05, min_md=0.05), and optionally runs a permutation pass (default max_perm=1000) to obtain empirical P-values fitted to a beta distribution for multiple-testing control. Decide whether to enable svQTL reporting and LD-based variant clustering (--ld, default 0 = no clustering) based on the analysis goal. Rationale: sQTLseekeR2 tests variants against a vector of splicing ratios, respecting the multivariate nature of alternative splicing rather than testing each event univariately.

## Related tools

- **sQTLseekeR2** (Core R engine performing genotype indexing, expression preprocessing, multivariate cis-association testing, covariate correction, and permutation-based multiple testing correction) — https://github.com/guigolab/sQTLseekeR2
- **Nextflow** (Workflow engine orchestrating the pipeline steps reproducibly (sqtlseeker2-nf)) — https://github.com/guigolab/sqtlseeker2-nf
- **Docker** (Container runtime used via -with-docker to pin the pipeline software environment)

## Examples

```
NXF_VER=22.04.0 ./nextflow run guigolab/sqtlseeker2-nf --genotype genotypes.tsv --tr_expr expression.tsv --metadata samples.tsv --gene_location genes.tsv --perm -with-docker
```

## Evaluation signals

- All four input files parse with the required columns (trId/geneId, indId/sampleId/group, chr/start/end/geneId) and the genotype file is sorted by coordinate before indexing
- Nominal pass completes with default cis window (win=5000 bp) and reported effect sizes pass the min_md=0.05 threshold
- Permuted pass yields empirical P-values with the expected beta-distribution fit (default max_perm=1000)
- sQTL calls are restricted to the specified FDR threshold (default 0.05) after permutation-based correction
- Pipeline runs reproducibly with the pinned Nextflow version (NXF_VER=22.04.0) inside Docker

## Limitations

- Whether paired sQTL/eQTL effects are caused by the same variant or two different variants in LD cannot be distinguished
- Co-localization and fine-mapping methods are not directly applicable within the multivariate, non-parametric framework
- LD-based variant clustering is disabled by default (--ld 0), so linked variants may be reported as separate associations unless clustering is enabled

## Evidence

- [readme] Pipeline: genotype indexing, expression preprocessing, nominal cis testing, permutation pass: "The pipeline performs the following analysis steps: Index the genotype file, Preprocess the transcript expression data, Test for association between splicing ratios and genetic variants in cis"
- [methods] Pipeline invocation with Docker: "./nextflow run guigolab/sqtlseeker2-nf -with-docker"
- [methods] Minimum effect size filter default: "--min_md MIN_MD             minimum effect size reported (default: 0.05)"
- [intro] Multivariate rationale for the sQTL test: "we propose an approach that takes into account the intrinsically multivariate nature of alternative splicing: variants are tested for association with a vector of AS phenotypes"
- [readme] sQTLseekeR2 enhancements relevant to execution: [evidence span withheld — anchor status: not_found]
