---
name: gapped-kmer-svm-sequence-classification
description: Use when the goal is to test whether sQTLs alter splicing by modifying
  RNA-binding protein binding sites, quantified as differences in |deltaSVM| between
  sQTLs and non-sQTL variants.
when_to_use_negative:
- When assessing disruption of splice-site consensus sequences rather than RBP binding
  motifs (the article treats these as distinct mechanisms)
- When variants do not overlap high-confidence eCLIP peaks (FDR < 0.01, fold-change
  >= 2) — deltaSVM scores are only interpretable within modeled binding sites
- When the RBP of interest has fewer than 30 top-5% sQTLs overlapping its eCLIP peaks
  or its model ROC AUC is below 0.8
tools:
- name: gkmSVM
  role: R package for training per-RBP gapped-kmer SVM classifiers and computing reference-minus-alternate
    deltaSVM scores
- name: sQTLseekeR2
  role: Produces the GTEx multi-tissue sQTL catalog whose top-5% variants are stratified
    against non-sQTLs for the deltaSVM comparison
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:7ba4ab70edab0fb2eac777edc75b3ca4f10b0785272adfbfb027dfda15650355
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sqtlseeker2-nf
  tools:
  - gkmSVM
  - sQTLseekeR2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gapped-kmer-svm-sequence-classification

## Summary

Trains per-RBP gapped-kmer SVM sequence models (gkmSVM R package) on high-confidence ENCODE eCLIP peaks, then computes deltaSVM scores for genetic variants to test whether sQTLs disrupt RNA-binding protein binding sites more strongly than non-sQTL variants.

## When to use

Use when the goal is to test whether sQTLs alter splicing by modifying RNA-binding protein binding sites, quantified as differences in |deltaSVM| between sQTLs and non-sQTL variants.

## When NOT to use

- When assessing disruption of splice-site consensus sequences rather than RBP binding motifs (the article treats these as distinct mechanisms)
- When variants do not overlap high-confidence eCLIP peaks (FDR < 0.01, fold-change >= 2) — deltaSVM scores are only interpretable within modeled binding sites
- When the RBP of interest has fewer than 30 top-5% sQTLs overlapping its eCLIP peaks or its model ROC AUC is below 0.8

## Inputs

- ENCODE eCLIP peak calls per RBP (filtered to FDR < 0.01, fold-change >= 2)
- sQTL catalog from sQTLseekeR2 applied to GTEx multi-tissue transcriptome data
- Genotypes with reference and alternate alleles for sQTL and non-sQTL variants
- Reference genome sequence (FASTA) for extracting variant-flanking sequence context

## Outputs

- Per-RBP gkmSVM classification models (retained if ROC AUC >= 0.8)
- deltaSVM scores (reference minus alternate allele SVM scores) for variants overlapping retained peaks
- rbp_deltaSVM_significance.tsv: rank-based test results with FDR-corrected significance per RBP
- Ranked list of the 10 significant RBPs, including RBFOX2, PRPF8, SF3B4 and PPIG

## How to apply

First filter ENCODE eCLIP peaks to high-confidence sites (FDR < 0.01, fold-change >= 2) and build positive/negative sequence sets for each RBP; train per-RBP gapped-kmer SVM classifiers with the gkmSVM R package and retain models reaching ROC AUC >= 0.8. Next, overlap the top-5% sQTLs from the sQTLseekeR2 catalog (GTEx, multi-tissue) with the retained eCLIP peaks and restrict to the 32 RBPs having at least 30 such sQTLs. For every variant overlapping the retained peaks, compute deltaSVM as the reference minus alternate allele SVM score, stratifying variants into sQTLs and non-sQTLs. Finally, test per RBP whether absolute deltaSVM values differ between sQTLs and non-sQTLs using a rank-based test with FDR correction at 0.05, and rank the significant RBPs.

## Related tools

- **gkmSVM** (R package for training per-RBP gapped-kmer SVM classifiers and computing reference-minus-alternate deltaSVM scores)
- **sQTLseekeR2** (Produces the GTEx multi-tissue sQTL catalog whose top-5% variants are stratified against non-sQTLs for the deltaSVM comparison) — https://github.com/guigolab/sqtlseeker2-nf

## Evaluation signals

- Exactly 10 RBPs pass the FDR < 0.05 threshold for the |deltaSVM| difference between sQTLs and non-sQTLs
- RBFOX2, PRPF8, SF3B4 and PPIG appear among the significant RBPs in rbp_deltaSVM_significance.tsv
- |deltaSVM| values are larger for sQTLs than non-sQTLs in 9 of the 10 significant proteins
- Only per-RBP models with ROC AUC >= 0.8 are retained (79 models expected)
- Per-RBP variant counts meet the minimum of 30 top-5% sQTLs overlapping eCLIP peaks before testing (32 RBPs retained)

## Limitations

- Only 79 of the trained RBP gkmSVM models reached ROC AUC >= 0.8, and only 32 RBPs had sufficient sQTL overlap, so conclusions are limited to those factors
- Interpretation of deltaSVM disruption depends on the quality of the eCLIP peaks; low-confidence peaks (FDR >= 0.01 or fold-change < 2) must be excluded
- Finding is correlative: significant |deltaSVM| differences support altered binding as a mechanism but do not prove causality for individual sQTLs

## Evidence

- [abstract] Many sQTLs also modify RBP binding sites: "While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins."
- [abstract] gkmSVM is the tool for gapped-kmer SVM: "gkmSVM: an R package for gapped-kmer SVM"
- [other] Ten RBPs significant at FDR < 0.05: [evidence span withheld — anchor status: not_found]
- [full_text] eCLIP peak filtering and deltaSVM workflow: [evidence span withheld — anchor status: not_found]
