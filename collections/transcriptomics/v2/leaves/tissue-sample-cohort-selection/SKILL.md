---
name: tissue-sample-cohort-selection
description: Use when preparing multi-tissue RNA-seq genotype and transcript expression
  matrices (e.g. GTEx V7) for sQTL mapping and you must decide which tissues/samples
  pass a minimum sample-size threshold for powered within-tissue QTL testing.
when_to_use_negative:
- Working with a single pre-defined tissue cohort whose sample size was already fixed
  by the study design — no per-tissue sample-size selection is needed
- Input expression/genotype data are not organized by tissue group (e.g. pooled cohorts
  or cell-line panels), so the tissue-wise >=70-sample criterion does not apply
- The downstream tool enforces different sample-size or tissue-criterion thresholds
  (e.g. GTEx V8 pipelines in broadinstitute/gtex-v8) — do not reuse the V7 48-tissue
  cutoff
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: sQTLseekeR2
  role: R package whose preprocessing requirements (transcript expression filters,
    genotype indexing) define the sample-size and filter thresholds applied after
    cohort selection
  repo: https://github.com/guigolab/sQTLseekeR2
- name: Nextflow
  role: Workflow engine that executes the sqtlseeker2-nf pipeline in which the tissue
    cohort selection and preprocessing step is embedded
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:7951a79e3442db79bb79ca211e64d34e1eb678ed375d7b653870719bf55ca5a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - Nextflow
  - sQTLseekeR2
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# tissue-sample-cohort-selection

## Summary

Selects GTEx V7 tissue sample cohorts for sQTL mapping by retaining only tissues with at least 70 samples, as required by the sqtlseeker2-nf preprocessing criterion, before applying sQTLseekeR2 gene, transcript and variant filters.

## When to use

Use when preparing multi-tissue RNA-seq genotype and transcript expression matrices (e.g. GTEx V7) for sQTL mapping and you must decide which tissues/samples pass a minimum sample-size threshold for powered within-tissue QTL testing.

## When NOT to use

- Working with a single pre-defined tissue cohort whose sample size was already fixed by the study design — no per-tissue sample-size selection is needed
- Input expression/genotype data are not organized by tissue group (e.g. pooled cohorts or cell-line panels), so the tissue-wise >=70-sample criterion does not apply
- The downstream tool enforces different sample-size or tissue-criterion thresholds (e.g. GTEx V8 pipelines in broadinstitute/gtex-v8) — do not reuse the V7 48-tissue cutoff

## Inputs

- GTEx V7 genotype matrix (TSV with chr, start, end, snpId; genotype codes 0=REF/REF, 1=REF/ALT, 2=ALT/ALT, -1=missing)
- Transcript expression TPM matrix (TSV with trId, geneId columns)
- Sample metadata table (indId, sampleId, group/tissue, covariates)
- Gene location table (chr, start, end, geneId)

## Outputs

- Per-tissue sample counts / retained-tissue list (tissues with >=70 samples)
- Subsetted sample metadata, expression, and genotype tables restricted to retained tissues
- Filtered variant set (TSV) and filtered gene/transcript expression set (TSV)

## How to apply

Load the four sqtlseeker2-nf input tables (genotype matrix with chr/start/end/snpId columns and alleles coded 0/1/2/-1, transcript TPM matrix with trId and geneId, sample metadata with indId/sampleId/group/covariates, and the gene location table). Compute sample counts per tissue/group and retain only tissues with at least 70 samples — for GTEx V7 this keeps 48 of 53 available tissues — subsetting samples consistently across metadata, expression, and genotype columns. After cohort selection, apply the downstream sQTLseekeR2 filters: keep genes with >=1 TPM in >=80% of samples and >=2 transcripts each at >=0.1 TPM, and biallelic variants with MAF >=0.01 and >=10 samples in each non-missing genotype group. Validate the result against the reported targets of 3,588,609 filtered variants and 16,010 filtered genes.

## Related tools

- **sQTLseekeR2** (R package whose preprocessing requirements (transcript expression filters, genotype indexing) define the sample-size and filter thresholds applied after cohort selection) — https://github.com/guigolab/sQTLseekeR2
- **Nextflow** (Workflow engine that executes the sqtlseeker2-nf pipeline in which the tissue cohort selection and preprocessing step is embedded) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
./nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- Retained-tissue count equals 48 out of the 53 available GTEx V7 tissues
- Every retained tissue has at least 70 samples in the sample metadata 'group' column
- Sample IDs in metadata, expression columns, and genotype columns are consistently restricted to the retained tissues (no orphan samples)
- Downstream filtered sets reproduce the reported targets of 3,588,609 variants and 16,010 genes
- Genotype codes remain in the expected domain (0/1/2 with -1 for missing) after subsetting

## Limitations

- The >=70-sample criterion trades tissue coverage for per-tissue sQTL power; 5 GTEx V7 tissues are excluded entirely
- Genotype-missingness coded as -1 must be excluded when counting samples per genotype group (>=10 per non-missing group), otherwise MAF/group-size filters are miscalculated
- Differences in preprocessing and statistical methodology can confound comparisons with other sQTL/eQTL catalogs, as noted for sGene vs eGene count differences in the article's discussion

## Evidence

- [other] 48-of-53 tissue retention under >=70-sample criterion: [evidence span withheld — anchor status: not_found]
- [other] validation targets for filtered sets: [evidence span withheld — anchor status: not_found]
- [readme] pipeline preprocessing step in sqtlseeker2-nf README: "The pipeline performs the following analysis steps:

* Index the genotype file
* Preprocess the transcript expression data"
- [discussion] reproducible sQTL mapping pipeline implementation: "our implementation of an enhanced pipeline for sQTL mapping based on sQTLseekeR2, Nextflow and Docker"
- [methods] pipeline invoked via nextflow: "./nextflow run guigolab/sqtlseeker2-nf -with-docker"
