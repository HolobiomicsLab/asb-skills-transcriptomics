---
name: eclip-peak-filtering
description: 'Use when testing whether sQTLs alter RNA-binding protein binding sites:
  you have ENCODE eCLIP peak calls, a sQTL catalog (e.g. sQTLseekeR2/GTEx), and need
  per-RBP SVM classifiers to score allelic binding impact.'
when_to_use_negative:
- When the question concerns splice-site sequence disruption rather than RBP binding-site
  alteration — the paper treats these as distinct mechanisms and finds many more variants
  affect RBP binding sites
- When eCLIP peaks lack FDR and fold-change information, since the high-confidence
  filter (FDR < 0.01, fold-change >= 2) cannot be applied
- When an RBP's eCLIP peaks overlap fewer than 30 top-5% sQTLs — such RBPs were excluded
  from the comparison in this analysis
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: gkmSVM
  role: R package for training per-RBP gapped-kmer SVM classifiers on filtered eCLIP
    peak sequences and computing reference/alternate allele scores for deltaSVM
- name: sQTLseekeR2
  role: Source of the multi-tissue sQTL catalog whose top-5% sQTLs are intersected
    with retained eCLIP peaks to define the 32 testable RBPs and the sQTL/non-sQTL
    stratification
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow pipeline that generated the sQTL catalog used as the variant strata
    for the deltaSVM comparison
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:5464043eebf1c091c8790b1002b0fac77844488f47f72caba5d7c9536dd75962
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - gkmSVM
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# eclip-peak-filtering

## Summary

Filters ENCODE eCLIP peaks to high-confidence RBP binding sites (FDR < 0.01, fold-change >= 2) and builds per-RBP positive/negative sequence sets for gkm-SVM modeling, enabling deltaSVM quantification of how sQTL alleles perturb RNA-binding protein sites versus non-sQTL variants.

## When to use

Use when testing whether sQTLs alter RNA-binding protein binding sites: you have ENCODE eCLIP peak calls, a sQTL catalog (e.g. sQTLseekeR2/GTEx), and need per-RBP SVM classifiers to score allelic binding impact.

## When NOT to use

- When the question concerns splice-site sequence disruption rather than RBP binding-site alteration — the paper treats these as distinct mechanisms and finds many more variants affect RBP binding sites
- When eCLIP peaks lack FDR and fold-change information, since the high-confidence filter (FDR < 0.01, fold-change >= 2) cannot be applied
- When an RBP's eCLIP peaks overlap fewer than 30 top-5% sQTLs — such RBPs were excluded from the comparison in this analysis

## Inputs

- ENCODE eCLIP peak calls (BED/narrowPeak with FDR and fold-change per RBP)
- Genome reference sequence for extracting peak and background sequences
- sQTL catalog from sQTLseekeR2 (GTEx multi-tissue, with top-5% sQTLs per RBP overlap)
- Variant set with reference and alternate alleles overlapping eCLIP peaks

## Outputs

- Per-RBP positive/negative training sequence sets from high-confidence peaks
- Per-RBP gkm-SVM models retained at ROC AUC >= 0.8 (79 models)
- Per-variant deltaSVM scores stratified by sQTL vs non-sQTL status
- Per-RBP significance table (rbp_deltaSVM_significance.tsv) ranking the 10 RBPs passing FDR < 0.05

## How to apply

Filter ENCODE eCLIP peaks per RBP to high-confidence sites (FDR < 0.01 and fold-change >= 2) and use them as positive sequences with matched background sequences as negatives. Train per-RBP gapped-kmer SVM classifiers with the gkmSVM R package and retain only models reaching ROC AUC >= 0.8 (79 models in this study). Restrict downstream testing to the RBPs whose retained peaks overlap at least 30 top-5% sQTLs from the sQTLseekeR2 catalog (32 RBPs qualified). Compute deltaSVM scores (reference minus alternate allele SVM scores) for all variants overlapping retained peaks, stratify them into sQTLs and non-sQTLs, and test per RBP whether absolute deltaSVM values differ using a rank-based test with FDR correction at 0.05.

## Related tools

- **gkmSVM** (R package for training per-RBP gapped-kmer SVM classifiers on filtered eCLIP peak sequences and computing reference/alternate allele scores for deltaSVM)
- **sQTLseekeR2** (Source of the multi-tissue sQTL catalog whose top-5% sQTLs are intersected with retained eCLIP peaks to define the 32 testable RBPs and the sQTL/non-sQTL stratification) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow pipeline that generated the sQTL catalog used as the variant strata for the deltaSVM comparison) — https://github.com/guigolab/sqtlseeker2-nf

## Evaluation signals

- Confirm that exactly 10 RBPs pass the FDR < 0.05 threshold and that RBFOX2, PRPF8, SF3B4 and PPIG appear among them in rbp_deltaSVM_significance.tsv
- Confirm 79 per-RBP gkm-SVM models are retained at ROC AUC >= 0.8 after training on the filtered peak sets
- Verify positive/negative sequence sets are built only from peaks passing FDR < 0.01 and fold-change >= 2
- Verify the downstream test set contains only the 32 RBPs with at least 30 top-5% sQTLs overlapping their eCLIP peaks
- Check that sQTLs show larger |deltaSVM| than non-sQTLs for 9 of the 10 significant RBPs

## Limitations

- Analysis is restricted to RBPs with at least 30 top-5% sQTLs overlapping retained eCLIP peaks (32 of 79 modeled RBPs qualified), so coverage of the RBP repertoire is limited by sQTL density
- Only SVM models reaching ROC AUC >= 0.8 are usable; RBPs with weaker classifiers cannot be assessed
- sQTL/eQTL co-localization and fine-mapping methods are not directly applicable within the multivariate, non-parametric sQTLseekeR2 framework, complicating attribution of overlapping effects

## Evidence

- [abstract] RBP binding sites carry most splicing-altering variation: "While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins."
- [abstract] gkmSVM is the modeling tool: [evidence span withheld — anchor status: not_found]
- [readme] sQTLseekeR2 provides the sQTL catalog: [evidence span withheld — anchor status: not_found]
