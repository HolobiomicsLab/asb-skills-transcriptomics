---
name: gtex-multitissue-sqtl-preprocessing
description: Use when preparing multi-tissue GTEx transcript expression (TPM) and
  genotype tables as sqtlseeker2-nf inputs, i.e., when the goal is cis-sQTL mapping
  and raw tissue, gene, transcript and variant sets must be filtered first.
when_to_use_negative:
- The input is already a filtered sQTLseekeR2-ready expression/genotype set — re-applying
  the tissue, gene and variant cutoffs would drop further data unnecessarily
- The goal is univariate eQTL mapping or LeafCutter-based intron excision-ratio analysis
  rather than multivariate transcript-proportion sQTL mapping, which uses different
  preprocessing
- Only a single tissue is analyzed and it has fewer than 70 samples, so the tissue
  sample-size criterion would remove everything
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3170
tools:
- name: sQTLseekeR2
  role: R package whose preprocessing of transcript expression and genotype indexing
    the filtering criteria feed into
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow pipeline that consumes the filtered genotype and expression TSVs
    for sQTL mapping
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: Nextflow
  role: Workflow engine used to run the reproducible sQTL mapping pipeline with Docker
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:b46a09ff1c370cca7bf082309ade91b26daffbd4283db941720b7d719d77969c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - Nextflow
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# gtex-multitissue-sqtl-preprocessing

## Summary

Preprocessing skill for GTEx-style multi-tissue sQTL mapping with sQTLseekeR2/sqtlseeker2-nf: retain tissues with >=70 samples, filter genes/transcripts on TPM-based criteria, and restrict variants to biallelic SNPs/indels with MAF >=0.01 and >=10 samples per genotype group.

## When to use

Use when preparing multi-tissue GTEx transcript expression (TPM) and genotype tables as sqtlseeker2-nf inputs, i.e., when the goal is cis-sQTL mapping and raw tissue, gene, transcript and variant sets must be filtered first.

## When NOT to use

- The input is already a filtered sQTLseekeR2-ready expression/genotype set — re-applying the tissue, gene and variant cutoffs would drop further data unnecessarily
- The goal is univariate eQTL mapping or LeafCutter-based intron excision-ratio analysis rather than multivariate transcript-proportion sQTL mapping, which uses different preprocessing
- Only a single tissue is analyzed and it has fewer than 70 samples, so the tissue sample-size criterion would remove everything

## Inputs

- GTEx V7 genotype matrix TSV (chr, start, end, snpId; genotype coded 0=REF/REF, 1=REF/ALT, 2=ALT/ALT, -1=missing)
- Transcript expression TPM matrix TSV (trId, geneId columns + sample TPM columns)
- Sample metadata table (indId, sampleId, group, covariates) with tissue group assignments
- Gene location/annotation table (chr, start, end, geneId)

## Outputs

- Filtered variant set TSV (biallelic, MAF >=0.01, >=10 samples per non-missing genotype group)
- Filtered gene/transcript expression TSV (genes with >=1 TPM in >=80% of samples and >=2 transcripts at >=0.1 TPM)
- Retained 48-tissue sample subsets feeding the sqtlseeker2-nf nominal association pass

## How to apply

Load the four sqtlseeker2-nf input tables: the genotype matrix (chr, start, end, snpId; alleles coded 0=REF/REF, 1=REF/ALT, 2=ALT/ALT, -1=missing), the transcript TPM matrix (trId, geneId), the sample metadata (indId, sampleId, group, covariates), and the gene location table (chr, start, end, geneId). Subset to tissues meeting the sample-size criterion of at least 70 samples (this retains 48 of 53 GTEx V7 tissues) and drop samples of excluded tissues from the metadata, expression, and genotype columns. Filter genes to those with >=1 TPM in >=80% of samples and at least 2 transcripts with isoform expression >=0.1 TPM, dropping failing genes/transcripts. Filter the genotype table to biallelic SNPs/indels with MAF >=0.01 and at least 10 samples in each non-missing genotype group (0/1/2, excluding -1). Write the filtered variant and gene/transcript expression sets as TSV files and confirm they reproduce the expected targets of ~3,588,609 variants and 16,010 genes.

## Related tools

- **sQTLseekeR2** (R package whose preprocessing of transcript expression and genotype indexing the filtering criteria feed into) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow pipeline that consumes the filtered genotype and expression TSVs for sQTL mapping) — https://github.com/guigolab/sqtlseeker2-nf
- **Nextflow** (Workflow engine used to run the reproducible sQTL mapping pipeline with Docker)

## Examples

```
nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- Variant count after filtering reproduces the reported target of 3,588,609 variants
- Gene count after filtering reproduces the reported target of 16,010 genes
- Exactly 48 of the 53 GTEx V7 tissues are retained, each with >=70 samples
- No gene in the filtered expression table has fewer than 2 transcripts with >=0.1 TPM isoform expression
- Every biallelic variant has at least 10 samples in each non-missing genotype group (0/1/2) and MAF >=0.01

## Limitations

- Differences in sGene vs eGene counts can be confounded by different preprocessing steps and statistical methodologies, so cross-comparison with eQTL catalogs is not straightforward
- The pipeline's LD-based variant clustering defaults to 0 (no clustering); LD-shared associations are not resolved unless --ld is set
- Samples with missing genotype (-1 coding) do not count toward the >=10-samples-per-genotype-group criterion and may leave sparse genotype groups

## Evidence

- [results] GTEx V7 sQTL mapping retained only 48 of the 53 available tissues, selecting those tissues with a sample size of at least 70 samples for sQTL analyses.: [evidence span withheld — anchor status: not_found]
- [methods] sqtlseeker2-nf ~ A pipeline for splicing QTL mapping: "sqtlseeker2-nf ~ A pipeline for splicing QTL mapping"
- [readme] The pipeline performs the following analysis steps: Index the genotype file; Preprocess the transcript expression data; Test for association between splicing ratios and genetic variants in cis (nominal pass): "The pipeline performs the following analysis steps: Index the genotype file Preprocess the transcript expression data Test for association between splicing ratios and genetic variants in cis"
- [readme] Faster pre-processing of transcript expression: [evidence span withheld — anchor status: external_doc]
- [abstract] We use it to analyze the GTEx dataset, generating a comprehensive catalog of sQTLs in the human genome.: "We use it to analyze the GTEx dataset, generating a comprehensive catalog of sQTLs in the human genome."
