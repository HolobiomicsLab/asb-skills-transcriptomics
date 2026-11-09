---
name: isoform-count-filtering
description: Use when preparing a transcript-level TPM expression matrix (e.g. GTEx
  V7 trId/geneId table) for multivariate sQTL mapping and you must keep only genes
  expressed with at least two isoforms.
when_to_use_negative:
- Input is single-isoform expression data or exon/junction counts (e.g. LeafCutter
  intron excision ratios) rather than a transcript-level TPM matrix
- The goal is gene-level eQTL mapping, where the >= 2 expressed isoforms requirement
  is unnecessary and would discard valid phenotypes
- Genes have already been filtered by the sQTLseekeR2 preprocessing step and re-filtering
  would double-apply thresholds
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: sQTLseekeR2
  role: R package whose preprocessing implements the gene/transcript expression filters
    applied to the TPM table before sQTL testing
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow pipeline that runs the expression preprocessing (including isoform-count
    filtering) prior to nominal and permuted sQTL passes
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: Nextflow
  role: Workflow engine used to execute the sqtlseeker2-nf pipeline reproducibly (with
    Docker)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:aa0aa6681bcecafe02b06cfb546f446cc48961fc30652b42915e9c7e1ecb08e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - Nextflow
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# isoform-count-filtering

## Summary

Preprocessing step of the sqtlseeker2-nf / sQTLseekeR2 workflow that restricts the GTEx V7 transcript expression (TPM) table to genes with sufficient multi-isoform expression before sQTL mapping, so only genes whose splicing can be tested are retained.

## When to use

Use when preparing a transcript-level TPM expression matrix (e.g. GTEx V7 trId/geneId table) for multivariate sQTL mapping and you must keep only genes expressed with at least two isoforms.

## When NOT to use

- Input is single-isoform expression data or exon/junction counts (e.g. LeafCutter intron excision ratios) rather than a transcript-level TPM matrix
- The goal is gene-level eQTL mapping, where the >= 2 expressed isoforms requirement is unnecessary and would discard valid phenotypes
- Genes have already been filtered by the sQTLseekeR2 preprocessing step and re-filtering would double-apply thresholds

## Inputs

- Transcript expression TPM matrix (trId, geneId columns; one column per sample) from GTEx V7
- Sample metadata table (indId, sampleId, group, covariates) restricted to the 48 tissues with >= 70 samples
- Gene location table (chr, start, end, geneId)

## Outputs

- Filtered gene/transcript expression table containing only genes with >= 2 transcripts expressed at >= 0.1 TPM and >= 1 TPM in >= 80% of samples (target: 16,010 genes), written as TSV

## How to apply

Load the transcript expression TPM matrix (columns trId, geneId) together with the sample metadata, and apply the sQTLseekeR2 default gene/transcript filters: keep genes with TPM >= 1 in at least 80% of samples, and among those keep only genes with at least 2 transcripts whose isoform expression is at least 0.1 TPM; drop failing genes and transcripts. Apply this after the tissue/sample-size subsetting (tissues with >= 70 samples retained 48 of 53 GTEx V7 tissues) and before writing the filtered expression set used for cis sQTL testing. The rationale is that sQTLseekeR2 tests a multivariate vector of transcript isoform proportions, so a gene must have at least two reliably expressed isoforms for the test to be defined. Applied across GTEx V7, this step contributes to reaching the reported target of 16,010 genes entering sQTL mapping.

## Related tools

- **sQTLseekeR2** (R package whose preprocessing implements the gene/transcript expression filters applied to the TPM table before sQTL testing) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow pipeline that runs the expression preprocessing (including isoform-count filtering) prior to nominal and permuted sQTL passes) — https://github.com/guigolab/sqtlseeker2-nf
- **Nextflow** (Workflow engine used to execute the sqtlseeker2-nf pipeline reproducibly (with Docker))

## Examples

```
./nextflow run guigolab/sqtlseeker2-nf -with-docker --gtf genes.gtf --tr_exp transcript_tpm.tsv --genotype genotypes.tsv --sample_meta samples.tsv
```

## Evaluation signals

- Filtered expression set reproduces the reported target of 16,010 genes on GTEx V7 inputs
- Every retained gene has at least 2 transcripts with isoform expression >= 0.1 TPM
- Every retained gene has TPM >= 1 in at least 80% of samples in the tissue subset
- Retained samples correspond exactly to the 48 GTEx V7 tissues with >= 70 samples (out of 53 available tissues)
- Output TSV preserves trId/geneId schema and contains no transcripts of dropped genes

## Limitations

- Filtering thresholds are sQTLseekeR2 defaults; different expression cutoffs may be needed for smaller datasets
- The difference in sGene vs eGene counts could be confounded by different preprocessing and statistical methodologies
- Genes with a single well-expressed isoform are excluded by design, since the multivariate splicing test requires at least two isoforms

## Evidence

- [other] Filter genes to those with >=1 TPM in >=80% of samples and >=2 transcripts whose isoform expression is at least 0.1 TPM: [evidence span withheld — anchor status: not_found]
- [readme] Preprocess the transcript expression data: "* Preprocess the transcript expression data"
- [discussion] we use a multivariate approach that monitors global changes in the relative abundances of a gene's transcript isoforms: [evidence span withheld — anchor status: not_found]
- [readme] Faster pre-processing of transcript expression: [evidence span withheld — anchor status: external_doc]
- [methods] sqtlseeker2-nf ~ A pipeline for splicing QTL mapping: "sqtlseeker2-nf ~ A pipeline for splicing QTL mapping"
