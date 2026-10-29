---
name: transcript-expression-tpm-thresholding
description: Use when a transcript-level TPM expression matrix (trId, geneId columns)
  is being prepared for sQTL mapping and unreliable genes/transcripts must be removed
  before association testing.
when_to_use_negative:
- The input is a gene-level count or CPM matrix rather than transcript-level TPM values
  with trId/geneId identifiers
- The goal is single-splicing-event analysis (e.g. LeafCutter intron excision ratios)
  rather than multivariate transcript isoform relative abundance testing with sQTLseekeR2
- Genes with a single dominant isoform are expected — the >=2 transcripts at >=0.1
  TPM filter would discard them by design
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3518
tools:
- name: sQTLseekeR2
  role: Defines the transcript expression preprocessing and the multivariate splicing
    phenotype (vector of transcript isoform relative abundances) that the TPM thresholds
    feed into
  repo: https://github.com/guigolab/sQTLseekeR2
- name: Nextflow
  role: Workflow engine executing the sqtlseeker2-nf pipeline, whose preprocessing
    step applies the expression filters
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:946f98e353821fc5ca4b8c20de296e5fc52ebcefc61f6d2f899391e70032be3d
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
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# transcript-expression-tpm-thresholding

## Summary

Applies the sQTLseekeR2/sqtlseeker2-nf preprocessing filters to GTEx V7 transcript expression TPM matrices so that only reliably quantified multi-isoform genes enter splicing QTL mapping. It reproduces the paper's filtered expression set of 16,010 genes.

## When to use

Use when a transcript-level TPM expression matrix (trId, geneId columns) is being prepared for sQTL mapping and unreliable genes/transcripts must be removed before association testing.

## When NOT to use

- The input is a gene-level count or CPM matrix rather than transcript-level TPM values with trId/geneId identifiers
- The goal is single-splicing-event analysis (e.g. LeafCutter intron excision ratios) rather than multivariate transcript isoform relative abundance testing with sQTLseekeR2
- Genes with a single dominant isoform are expected — the >=2 transcripts at >=0.1 TPM filter would discard them by design

## Inputs

- Transcript expression TPM matrix with trId and geneId columns (GTEx V7)
- Sample metadata table (indId, sampleId, group, covariates) to define the analyzed sample set

## Outputs

- Filtered gene/transcript expression table (>=1 TPM in >=80% of samples, >=2 transcripts at >=0.1 TPM) as TSV
- Set of retained genes validated against the expected 16,010-gene target

## How to apply

Load the transcript expression TPM table with columns trId and geneId in pandas. For each gene, require at least 1 TPM in at least 80% of samples, and require the gene to have at least 2 transcripts whose isoform expression reaches at least 0.1 TPM. Drop genes and transcripts failing these thresholds from the expression table while keeping the matching samples/tissues (those with >=70 samples in GTEx V7, yielding 48 retained tissues). Write the filtered gene/transcript expression set as a TSV and validate that it reproduces the paper's target of 16,010 genes. The multi-isoform requirement matters because sQTLseekeR2 tests variants against a vector of transcript isoform relative abundances, so single-isoform genes cannot yield testable splicing phenotypes.

## Related tools

- **sQTLseekeR2** (Defines the transcript expression preprocessing and the multivariate splicing phenotype (vector of transcript isoform relative abundances) that the TPM thresholds feed into) — https://github.com/guigolab/sQTLseekeR2
- **Nextflow** (Workflow engine executing the sqtlseeker2-nf pipeline, whose preprocessing step applies the expression filters) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
keep = (tpm_df[tpm_cols] >= 1).mean(axis=1) >= 0.8; n_iso = (tpm_df[tpm_cols] >= 0.1).sum(axis=1); filtered = tpm_df[keep & (n_iso >= 2)]
```

## Evaluation signals

- Filtered expression set reproduces the reported target of 16,010 genes when run on GTEx V7 inputs
- Every retained gene has >=2 transcripts with isoform expression >=0.1 TPM and no transcripts below threshold remain
- Every retained gene shows >=1 TPM in at least 80% of the samples of its tissue group
- Only samples from the 48 GTEx V7 tissues with >=70 samples are present in the filtered expression columns

## Limitations

- Filtering is coupled to the GTEx V7 sample-size criterion (>=70 samples, 48 of 53 tissues); other cohorts may need re-tuned thresholds
- The expression filter cannot distinguish biological low expression from quantification noise in the RSEM TPM estimates
- Differences in preprocessing and statistical methodology can confound downstream comparisons, e.g. sGene vs eGene counts

## Evidence

- [methods] Filter genes to those with >=1 TPM in >=80% of samples and >=2 transcripts whose isoform expression is at least 0.1 TPM: [evidence span withheld — anchor status: not_found]
- [methods] confirm the filtered sets reproduce the reported targets of 3,588,609 variants and 16,010 genes: [evidence span withheld — anchor status: not_found]
- [readme] Faster pre-processing of transcript expression: [evidence span withheld — anchor status: external_doc]
- [discussion] we use a multivariate approach that monitors global changes in the relative abundances of a gene's transcript isoforms: [evidence span withheld — anchor status: not_found]
