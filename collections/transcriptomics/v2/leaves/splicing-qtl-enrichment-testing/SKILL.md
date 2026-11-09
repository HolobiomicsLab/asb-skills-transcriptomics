---
name: splicing-qtl-enrichment-testing
description: Use when you have an sQTL catalog (e.g. from sQTLseekeR2 on GTEx) plus
  RBP eCLIP peaks and want to test whether sQTLs change RBP binding motifs more than
  matched non-sQTL variants.
when_to_use_negative:
- Your goal is to test enrichment of sQTLs in splice-site sequences rather than RBP
  binding sites — the article treats these as distinct mechanism classes
- You lack eCLIP peaks for the RBP of interest, so no gkm-SVM model can be trained
  or scored
- Variants come from a univariate splicing QTL analysis not accounting for the multivariate
  nature of alternative splicing (no LeafCutter intron excision ratios), making the
  sQTL/non-sQTL stratification non-comparable
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: gkmSVM
  role: R package used to train per-RBP gapped-kmer SVM classifiers and compute deltaSVM
    scores for reference vs alternate alleles
- name: sQTLseekeR2
  role: R package/pipeline used to map the sQTL catalog whose variants are stratified
    for the deltaSVM enrichment test
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: LeafCutter
  role: Annotation-free quantification of intron excision ratios used as the multivariate
    splicing phenotype for sQTL mapping
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:3a1d2ad2d054a7084e12533bae9a5e33b0ff9d821f14f6b8d9bed2eeeb24c1de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sqtlseeker2-nf
  tools:
  - LeafCutter
  - gkmSVM
  - sQTLseekeR2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# splicing-qtl-enrichment-testing

## Summary

Tests whether splicing QTLs (sQTLs) disproportionately alter RNA-binding protein (RBP) binding sites relative to non-sQTL variants, by comparing |deltaSVM| scores from per-RBP gapped-kmer SVM models. Grounds the finding that more splicing variants modify RBP binding sites than splice-site sequences.

## When to use

Use when you have an sQTL catalog (e.g. from sQTLseekeR2 on GTEx) plus RBP eCLIP peaks and want to test whether sQTLs change RBP binding motifs more than matched non-sQTL variants.

## When NOT to use

- Your goal is to test enrichment of sQTLs in splice-site sequences rather than RBP binding sites — the article treats these as distinct mechanism classes
- You lack eCLIP peaks for the RBP of interest, so no gkm-SVM model can be trained or scored
- Variants come from a univariate splicing QTL analysis not accounting for the multivariate nature of alternative splicing (no LeafCutter intron excision ratios), making the sQTL/non-sQTL stratification non-comparable

## Inputs

- sQTL catalog from sQTLseekeR2 (sqtlseeker2-nf pipeline on GTEx multi-tissue data)
- ENCODE eCLIP peaks per RBP with FDR and fold-change annotations
- Genome sequences flanking each variant (reference and alternate alleles)
- GTEx genotype files and intron excision ratios quantified with LeafCutter

## Outputs

- Per-RBP ROC AUC values for the gkm-SVM classifiers (retained at >= 0.8)
- deltaSVM scores (reference minus alternate allele) for variants overlapping eCLIP peaks
- Table of RBPs passing the FDR < 0.05 rank-based test (e.g. rbp_deltaSVM_significance.tsv) with 10 significant RBPs including RBFOX2, PRPF8, SF3B4 and PPIG

## How to apply

Filter ENCODE eCLIP peaks to high-confidence sites (FDR < 0.01, fold-change >= 2) and build positive/negative sequence sets per RBP. Train per-RBP gapped-kmer SVM classifiers with the gkmSVM R package, retaining models with ROC AUC >= 0.8 (79 of them in the reference analysis). Restrict to the 32 RBPs having at least 30 top-5% sQTLs from the sQTLseekeR2 catalog overlapping their eCLIP peaks. Compute deltaSVM scores (reference minus alternate allele SVM scores) for all variants overlapping retained peaks, stratify into sQTLs vs non-sQTLs, and test per RBP whether absolute deltaSVM values differ using a rank-based test with FDR correction at 0.05. The rationale is that sQTLs enriched in large |deltaSVM| scores implicate the corresponding RBP in splicing regulation.

## Related tools

- **gkmSVM** (R package used to train per-RBP gapped-kmer SVM classifiers and compute deltaSVM scores for reference vs alternate alleles)
- **sQTLseekeR2** (R package/pipeline used to map the sQTL catalog whose variants are stratified for the deltaSVM enrichment test) — https://github.com/guigolab/sqtlseeker2-nf
- **LeafCutter** (Annotation-free quantification of intron excision ratios used as the multivariate splicing phenotype for sQTL mapping)

## Examples

```
nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- Exactly 10 RBPs pass the FDR < 0.05 threshold for the |deltaSVM| rank-based test
- RBFOX2, PRPF8, SF3B4 and PPIG appear among the significant RBPs in rbp_deltaSVM_significance.tsv
- Retained gkm-SVM models all reach ROC AUC >= 0.8 (79 models)
- Only RBPs with at least 30 top-5% sQTLs overlapping their eCLIP peaks (32 RBPs) are tested
- For 9 of the 10 significant RBPs, sQTLs show larger |deltaSVM| values than non-sQTLs

## Limitations

- Results depend on the quality of eCLIP peak calls; only high-confidence peaks (FDR < 0.01, fold-change >= 2) are used
- The analysis is restricted to 32 RBPs because most RBPs lack the required >= 30 overlapping top-5% sQTLs
- Cannot distinguish whether paired sQTL/eQTL effects are caused by the same variant or two different variants in LD
- Co-localization and fine-mapping methods are not directly applicable within the multivariate, non-parametric sQTLseekeR2 framework

## Evidence

- [abstract] While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins.: "While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins."
- [abstract] gkmSVM R package: "gkmSVM: an R package for gapped-kmer SVM"
- [intro] we have developed a pipeline for efficient and reproducible sQTL mapping: "we have developed a pipeline for efficient and reproducible sQTL mapping"
- [intro] the intron excision ratios of an intron cluster obtained by LeafCutter18: "the intron excision ratios of an intron cluster obtained by LeafCutter18"
- [readme] Test for association between splicing ratios and genetic variants in cis (nominal pass): "Test for association between splicing ratios and genetic variants in *cis* (nominal pass)"
