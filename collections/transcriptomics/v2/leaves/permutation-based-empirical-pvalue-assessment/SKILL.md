---
name: permutation-based-empirical-pvalue-assessment
description: Use when mapping cis-sQTLs on transcript-ratio phenotypes and you need
  FDR-controlled significance calls — i.e., after a nominal pass you require permutation-calibrated
  empirical P-values at a target FDR such as 0.05.
when_to_use_negative:
- The goal is a plain eQTL univariate-expression scan where standard linear-model
  P-values and Bonferroni/BH correction suffice rather than a multivariate splicing
  phenotype
- Only exploratory nominal association statistics are needed and the cost of a 1000-permutation
  pass is unwarranted
- Cis windows were generated without LD-aware variant structure and downstream co-localization/fine-mapping
  is expected, since the multivariate non-parametric framework does not support those
  methods directly
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2269
tools:
- name: sQTLseekeR2
  role: R package executing the nominal and permutation association tests and empirical
    P-value calculation for multivariate splicing phenotypes
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow workflow wrapping sQTLseekeR2, exposing --mode permuted, --max_perm,
    --kp and --fdr parameters for reproducible permutation passes in Docker
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: Nextflow
  role: Workflow engine that orchestrates the nominal and permuted passes reproducibly
    across tissue groups
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:47c41e66746cb141831e1aee155e00134bef72183b3ad5560b5867425d0433b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - Docker
  - Nextflow
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Permutation-based empirical P-value assessment

## Summary

An sQTLseekeR2/sqtlseeker2-nf analysis step that assigns each gene an empirical P-value via an adaptive permutation pass after the nominal cis-sQTL scan, enabling multiple-testing correction (FDR) for multivariate splicing phenotypes.

## When to use

Use when mapping cis-sQTLs on transcript-ratio phenotypes and you need FDR-controlled significance calls — i.e., after a nominal pass you require permutation-calibrated empirical P-values at a target FDR such as 0.05.

## When NOT to use

- The goal is a plain eQTL univariate-expression scan where standard linear-model P-values and Bonferroni/BH correction suffice rather than a multivariate splicing phenotype
- Only exploratory nominal association statistics are needed and the cost of a 1000-permutation pass is unwarranted
- Cis windows were generated without LD-aware variant structure and downstream co-localization/fine-mapping is expected, since the multivariate non-parametric framework does not support those methods directly

## Inputs

- Genotype file sorted by coordinate (0/1/2 coding, -1 missing), indexed by the pipeline
- Preprocessed transcript expression file with trId and geneId columns (transcript ratios per gene)
- Metadata file with indId, sampleId and group columns plus covariates (covariate regression enabled with --covariates true)
- Gene location file with chr, start, end and geneId columns
- Nominal-pass output (all-tests.nominal.tsv) driving the adaptive permutation pass

## Outputs

- all-tests.permuted.tsv containing per-phenotype empirical P-values from permutations
- sqtls-0.05fdr.permuted.tsv listing significant sQTLs with the fdr column computed across empirical P-values at the 0.05 threshold
- Per-group result files under result/groups/<group>/ for aggregation across tissues

## How to apply

Run the nominal association pass of sQTLseekeR2 (via the sqtlseeker2-nf Nextflow pipeline) testing variants in cis against a vector of transcript isoform ratios per gene. Then set --mode permuted with --max_perm 1000 so the adaptive permutation pass reruns genes with promising nominal statistics, generating empirical P-values from genotype permutations; batch genes using --kp (default 10 genes per batch in the permuted pass). Compute the fdr column across the empirical P-values in all-tests.permuted.tsv at --fdr 0.05 (optionally applying --min_md 0.05 as a minimum effect size filter) to emit sqtls-0.05fdr.permuted.tsv. The rationale is that splicing phenotypes are intrinsically multivariate, so permutation provides a robust null distribution for the non-parametric framework where closed-form multiple-testing correction is inadequate.

## Related tools

- **sQTLseekeR2** (R package executing the nominal and permutation association tests and empirical P-value calculation for multivariate splicing phenotypes) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow workflow wrapping sQTLseekeR2, exposing --mode permuted, --max_perm, --kp and --fdr parameters for reproducible permutation passes in Docker) — https://github.com/guigolab/sqtlseeker2-nf
- **Nextflow** (Workflow engine that orchestrates the nominal and permuted passes reproducibly across tissue groups) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
NXF_VER=22.04.0 nextflow run guigolab/sqtlseeker2-nf --mode permuted --max_perm 1000 --covariates true --fdr 0.05 --min_md 0.05 -with-docker
```

## Evaluation signals

- all-tests.permuted.tsv exists for every tissue group and contains an empirical P-value column for each tested phenotype
- sqtls-0.05fdr.permuted.tsv exists per group and its fdr column is computed across empirical P-values with all rows <= 0.05
- Number of distinct sGenes in the aggregated catalog is consistent with the reported GTEx V7 catalog (6,963 sGenes: 6,685 protein-coding genes and 278 lincRNAs)
- Aggregated per-tissue significant sQTL totals reproduce the reported GTEx V7 catalog counts (210,485 cis sQTLs)
- Pipeline completed with --max_perm 1000 without genotyping or transcript-expression preprocessing errors (files indexed and preprocessed successfully)

## Limitations

- The permutation pass is optional and computationally heavier; the pipeline README notes it as '(permutation pass, optional)'
- LD-based clustering (--ld threshold, default 0) and co-localization/fine-mapping methods are not directly applicable within the multivariate, non-parametric framework, limiting fine-mapping of permuted hits
- Whether paired sQTL/eQTL effects come from the same variant or two variants in LD cannot be distinguished from these results
- Nextflow DSL1 is no longer supported in recent releases, so the workflow must be run with a pinned NXF_VER (e.g. 22.04.0)

## Evidence

- [readme] Obtain an empirical P-value for each phenotype (permutation pass, optional): "Obtain an empirical P-value for each phenotype (permutation pass, optional)"
- [readme] Permutation-based schema for multiple testing correction: "Permutation-based schema for multiple testing correction "
- [methods] number of genes per batch in permuted pass (default: 10): "--kp KP                     number of genes per batch in permuted pass (default: 10)"
- [readme] Test for association between splicing ratios and genetic variants in cis (nominal pass): "Test for association between splicing ratios and genetic variants in *cis* (nominal pass)"
