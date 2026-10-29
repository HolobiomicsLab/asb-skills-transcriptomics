---
name: delta-svm-variant-scoring
description: Use when sQTL (or other regulatory) variants need to be assessed for
  whether they alter RBP binding sites rather than splice-site consensus sequences,
  using per-RBP sequence classifiers and deltaSVM effect scores.
when_to_use_negative:
- When the goal is multivariate sQTL discovery itself (mapping sQTLs from intron excision
  ratios) rather than downstream functional annotation of an existing sQTL catalog
- When variants of interest lie outside high-confidence eCLIP peaks (FDR < 0.01, fold-change
  >= 2), since deltaSVM models are trained and applied on RBP binding sites
- When an RBP has fewer than 30 top-5% sQTLs overlapping its eCLIP peaks, as it is
  excluded from the differential test
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0654
tools:
- name: gkmSVM
  role: R package for training per-RBP gapped-kmer SVM classifiers and computing reference/alternate
    allele SVM scores for deltaSVM
- name: sQTLseekeR2
  role: Produces the GTEx sQTL catalog (top-5% sQTLs) that supplies the sQTL variant
    set stratified against non-sQTLs
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow/Docker pipeline that generates the sQTL catalog used to stratify
    variants into sQTLs and non-sQTLs
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:169f82e9bed813f4fe9f22521976727a32d091f9cfe30482fde798bfb8fca19f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - gkmSVM
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# delta-svm-variant-scoring

## Summary

Scores genetic variants for allele-specific impact on RNA-binding protein (RBP) motif landscapes by training per-RBP gapped-kmer SVMs (gkmSVM R package) and computing deltaSVM (reference minus alternate allele SVM scores), then testing whether sQTLs show larger |deltaSVM| than non-sQTL variants.

## When to use

Use when sQTL (or other regulatory) variants need to be assessed for whether they alter RBP binding sites rather than splice-site consensus sequences, using per-RBP sequence classifiers and deltaSVM effect scores.

## When NOT to use

- When the goal is multivariate sQTL discovery itself (mapping sQTLs from intron excision ratios) rather than downstream functional annotation of an existing sQTL catalog
- When variants of interest lie outside high-confidence eCLIP peaks (FDR < 0.01, fold-change >= 2), since deltaSVM models are trained and applied on RBP binding sites
- When an RBP has fewer than 30 top-5% sQTLs overlapping its eCLIP peaks, as it is excluded from the differential test

## Inputs

- ENCODE eCLIP peak BED/NarrowPeak per RBP, filtered to FDR < 0.01 and fold-change >= 2
- Reference genome FASTA for extracting positive/negative sequence sets around eCLIP peaks
- sQTL catalog from sQTLseekeR2 (GTEx multi-tissue), with top-5% sQTL coordinates
- Variant VCF with reference and alternate alleles overlapping retained eCLIP peaks

## Outputs

- Per-RBP gkm-SVM models with ROC AUC metrics (retained threshold >= 0.8)
- deltaSVM scores table (reference minus alternate allele SVM score per variant, stratified into sQTLs vs non-sQTLs)
- rbp_deltaSVM_significance.tsv ranking the 10 RBPs significant at FDR < 0.05

## How to apply

First filter ENCODE eCLIP peaks to high-confidence sites (FDR < 0.01, fold-change >= 2) and build positive/negative FASTA sequence sets for each RBP, then train per-RBP gapped-kmer SVM classifiers with the gkmSVM R package, retaining only models reaching ROC AUC >= 0.8 (79 models retained). Restrict analysis to RBPs with at least 30 top-5% sQTLs from the sQTLseekeR2 catalog (produced by the sqtlseeker2-nf Nextflow/Docker pipeline on GTEx) overlapping their eCLIP peaks, yielding 32 assessable RBPs. Compute deltaSVM scores as the reference minus alternate allele SVM scores for every variant overlapping the retained peaks, stratifying variants into sQTLs and non-sQTLs. For each RBP, test whether absolute deltaSVM values differ between sQTLs and non-sQTLs using a rank-based test with FDR correction at 0.05; ten RBPs (including RBFOX2, PRPF8, SF3B4 and PPIG) are expected to be significant, with larger |deltaSVM| for sQTLs in nine of them.

## Related tools

- **gkmSVM** (R package for training per-RBP gapped-kmer SVM classifiers and computing reference/alternate allele SVM scores for deltaSVM)
- **sQTLseekeR2** (Produces the GTEx sQTL catalog (top-5% sQTLs) that supplies the sQTL variant set stratified against non-sQTLs) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow/Docker pipeline that generates the sQTL catalog used to stratify variants into sQTLs and non-sQTLs) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
./nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- Exactly 10 RBPs pass the FDR < 0.05 threshold in the rank-based differential test
- RBFOX2, PRPF8, SF3B4 and PPIG appear among the significant RBPs in rbp_deltaSVM_significance.tsv
- The number of retained RBP gkm-SVM models with ROC AUC >= 0.8 equals 79
- The number of assessed RBPs with at least 30 top-5% sQTLs overlapping their eCLIP peaks equals 32
- Nine of the ten significant RBPs show larger |deltaSVM| values for sQTLs than for non-sQTLs

## Limitations

- Only 32 of the RBP models are assessable because the test requires at least 30 top-5% sQTLs overlapping the RBP's eCLIP peaks, so RBPs with sparse sQTL overlap are missed
- deltaSVM reflects predicted motif/sequence impact, not measured RBP binding, so significant differences indicate altered binding-site potential rather than direct binding evidence

## Evidence

- [abstract] While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins.: "While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins."
- [abstract] gkmSVM: an R package for gapped-kmer SVM: "gkmSVM: an R package for gapped-kmer SVM"
- [methods] ./nextflow run guigolab/sqtlseeker2-nf -with-docker: "./nextflow run guigolab/sqtlseeker2-nf -with-docker"
- [readme] Permutation-based schema for multiple testing correction: [evidence span withheld — anchor status: external_doc]
