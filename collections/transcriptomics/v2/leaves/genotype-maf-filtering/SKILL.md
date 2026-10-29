---
name: genotype-maf-filtering
description: Use when a raw genotype matrix (columns chr, start, end, snpId; alleles
  coded 0=REF/REF, 1=REF/ALT, 2=ALT/ALT, -1=missing) must be filtered to well-powered
  biallelic variants before sQTL association testing.
when_to_use_negative:
- The variant table is already the filtered sQTLseekeR2 input (e.g. it already contains
  only the ~3.6M passing biallelic variants)
- The goal is a single-tissue eQTL/expression analysis where MAF and per-genotype-group
  counts were set by a different consortium standard (e.g. GTEx V8 eQTL filters)
- The genotype data are multiallelic or structural variants that must be decomposed/recodeed
  before dosage-based filtering applies
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: sQTLseekeR2
  role: R package whose default variant filters (biallelic, MAF and genotype-group
    count thresholds) this skill reproduces prior to indexing and cis sQTL testing
  repo: https://github.com/guigolab/sQTLseekeR2
- name: Nextflow
  role: Workflow engine running the sqtlseeker2-nf pipeline in which the filtered
    genotype table is the input to the 'Index the genotype file' step
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:f52773224f899f6afaa6182c0ee22f4b1b1c84db57d4a4fa3c24dfc1eb426ae5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - Nextflow
  - sQTLseekeR2
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# genotype-maf-filtering

## Summary

Preprocesses genotype matrices for sQTL mapping by retaining only biallelic SNPs/indels with minor allele frequency >= 0.01 and sufficient per-genotype-group sample counts, reproducing the sQTLseekeR2 default variant filters applied to GTEx V7 in the sqtlseeker2-nf pipeline.

## When to use

Use when a raw genotype matrix (columns chr, start, end, snpId; alleles coded 0=REF/REF, 1=REF/ALT, 2=ALT/ALT, -1=missing) must be filtered to well-powered biallelic variants before sQTL association testing.

## When NOT to use

- The variant table is already the filtered sQTLseekeR2 input (e.g. it already contains only the ~3.6M passing biallelic variants)
- The goal is a single-tissue eQTL/expression analysis where MAF and per-genotype-group counts were set by a different consortium standard (e.g. GTEx V8 eQTL filters)
- The genotype data are multiallelic or structural variants that must be decomposed/recodeed before dosage-based filtering applies

## Inputs

- Genotype matrix TSV with leading chr, start, end, snpId columns and per-sample allele dosages coded 0=REF/REF, 1=REF/ALT, 2=ALT/ALT, -1=missing
- Sample metadata table (indId, sampleId, group, covariates) restricted to the 48 retained GTEx V7 tissues (>= 70 samples)

## Outputs

- Filtered variant set TSV (~3,588,609 variants) restricted to biallelic SNPs/indels with MAF >= 0.01 and >= 10 samples per non-missing genotype group
- Indexed genotype file consumed by sQTLseekeR2 for cis association testing

## How to apply

Load the genotype table in pandas, keeping the leading chr, start, end, snpId columns and treating 0, 1, and 2 as REF/REF, REF/ALT, and ALT/ALT dosage calls with -1 encoding missing. Retain only biallelic SNPs/indels with MAF >= 0.01 computed over non-missing samples. Additionally require at least 10 samples in each non-missing genotype group (0/1/2, excluding -1) so that no dosage class is too sparse for the multivariate test. Write the filtered variant set as a TSV; validation targets reproducing the reported 3,588,609 variants from GTEx V7.

## Related tools

- **sQTLseekeR2** (R package whose default variant filters (biallelic, MAF and genotype-group count thresholds) this skill reproduces prior to indexing and cis sQTL testing) — https://github.com/guigolab/sQTLseekeR2
- **Nextflow** (Workflow engine running the sqtlseeker2-nf pipeline in which the filtered genotype table is the input to the 'Index the genotype file' step) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
import pandas as pd; gt = pd.read_csv('gtex_genotypes.tsv', sep='\t'); maf = gt.iloc[:,4:].apply(lambda c: c[c>=0].value_counts(normalize=True).min(), axis=1); gt[(gt.iloc[:,4:] != -1).sum(axis=1) > 0].loc[maf >= 0.01].to_csv('filtered_genotypes.tsv', sep='\t', index=False)
```

## Evaluation signals

- Filtered variant count matches the reported target of 3,588,609 variants when run on the GTEx V7 inputs
- Every retained variant is biallelic and has MAF >= 0.01 over non-missing samples
- For each retained variant, all non-missing genotype groups (0/1/2, excluding -1) contain >= 10 samples
- No retained variant contains dosage values outside {0, 1, 2} or a -1 that was not treated as missing
- The filtered TSV preserves the expected schema: chr, start, end, snpId followed by one dosage column per sample of the 48 retained tissues

## Limitations

- The LD-based variant clustering option in sQTLseekeR2 (default: 0, no clustering) is a separate post-filtering step and is not covered by this skill
- Filter thresholds are hardcoded to the sQTLseekeR2 defaults used for GTEx V7; other studies may require different MAF or per-group count cutoffs
- The skill assumes the tissue/sample restriction (48 tissues with >= 70 samples) has already been applied to the genotype columns

## Evidence

- [other] Filter the genotype table to biallelic SNPs/indels with MAF >= 0.01 and at least 10 samples in each non-missing genotype group (0/1/2, excluding -1).: [evidence span withheld — anchor status: not_found]
- [other] Validation: confirm the filtered sets reproduce the reported targets of 3,588,609 variants and 16,010 genes.: [evidence span withheld — anchor status: not_found]
- [other] GTEx V7 sQTL mapping retained only 48 of the 53 available tissues, selecting those tissues with a sample size of at least 70 samples for sQTL analyses.: [evidence span withheld — anchor status: not_found]
- [readme] Index the genotype file. `index.genotype` compresses and indexes the genotype file to optimize the: [evidence span withheld — anchor status: external_doc]
