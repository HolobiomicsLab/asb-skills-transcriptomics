---
name: rbp-binding-site-disruption-analysis
description: Use when you have a catalog of splicing QTLs (e.g. from sQTLseekeR2 on
  GTEx) plus ENCODE eCLIP peak sets and want to identify which RBPs' binding sites
  are altered by sQTLs relative to non-sQTL variants.
when_to_use_negative:
- You need cis-association mapping itself rather than downstream interpretation of
  an existing sQTL catalog — use sqtlseeker2-nf/sQTLseekeR2 directly for that step.
- Your variants do not overlap high-confidence eCLIP peaks (FDR < 0.01, FC >= 2) or
  the RBP has fewer than 30 top-5% sQTLs in its peaks — the paper restricted analysis
  to 32 such RBPs.
- You are evaluating protein-coding loss-of-function or splice-site variants — tools
  like LOFTEE address that complementary question, not RBP motif disruption.
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: gkmSVM
  role: R package for gapped-kmer SVM used to train per-RBP classifiers on eCLIP peak
    sequences and compute deltaSVM allele-effect scores
- name: sQTLseekeR2
  role: R package (run via the sqtlseeker2-nf Nextflow pipeline) producing the multivariate
    sQTL catalog whose variants are stratified for deltaSVM comparison
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: LeafCutter
  role: Annotation-free quantification of intron excision ratios providing the splicing
    phenotypes upstream of sQTL mapping
- name: Nextflow
  role: Workflow engine for the reproducible sqtlseeker2-nf sQTL mapping pipeline
    (with Docker)
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:94a138981d4f253bcb67911a93ad4beceab8d169292a36abd5591f29b1e24dee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sqtlseeker2-nf
  tools:
  - LeafCutter
  - Nextflow
  - gkmSVM
  - sQTLseekeR2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# rbp-binding-site-disruption-analysis

## Summary

Reproduces the sQTL-downstream analysis from Garrido-Martín et al. testing whether splicing QTLs disrupt RNA-binding protein (RBP) binding sites more than splice-site motifs, by training gapped-kmer SVM (gkmSVM) models on ENCODE eCLIP peaks and comparing deltaSVM scores of sQTLs vs non-sQTLs.

## When to use

Use when you have a catalog of splicing QTLs (e.g. from sQTLseekeR2 on GTEx) plus ENCODE eCLIP peak sets and want to identify which RBPs' binding sites are altered by sQTLs relative to non-sQTL variants.

## When NOT to use

- You need cis-association mapping itself rather than downstream interpretation of an existing sQTL catalog — use sqtlseeker2-nf/sQTLseekeR2 directly for that step.
- Your variants do not overlap high-confidence eCLIP peaks (FDR < 0.01, FC >= 2) or the RBP has fewer than 30 top-5% sQTLs in its peaks — the paper restricted analysis to 32 such RBPs.
- You are evaluating protein-coding loss-of-function or splice-site variants — tools like LOFTEE address that complementary question, not RBP motif disruption.

## Inputs

- ENCODE eCLIP peak BED files per RBP (filtered to FDR < 0.01, fold-change >= 2)
- sQTL catalog from sQTLseekeR2 (sqtlseeker2-nf output on GTEx v8)
- Genotypes (VCF) for sQTL and non-sQTL variants overlapping eCLIP peaks
- Reference genome sequence (FASTA) for extracting variant-flanking sequences

## Outputs

- Per-RBP gkmSVM classifier models with ROC AUC >= 0.8
- deltaSVM scores (reference minus alternate allele) per variant per RBP
- rbp_deltaSVM_significance.tsv ranking the 10 RBPs significant at FDR < 0.05 (incl. RBFOX2, PRPF8, SF3B4, PPIG)

## How to apply

First obtain the sQTL catalog by running the sqtlseeker2-nf Nextflow pipeline (sQTLseekeR2 inside Docker) on GTEx intron excision ratios from LeafCutter. For each RBP, filter ENCODE eCLIP peaks to high-confidence sites (FDR < 0.01, fold-change >= 2), build positive/negative sequence sets, and train gapped-kmer SVM classifiers with the gkmSVM R package, retaining only the models reaching ROC AUC >= 0.8 (79 of them in the paper). Restrict testing to RBPs whose eCLIP peaks overlap at least 30 top-5% sQTLs from the sQTLseekeR2 catalog (32 RBPs in the paper). Compute deltaSVM scores (reference minus alternate allele SVM scores) for all variants overlapping retained peaks, stratify into sQTLs vs non-sQTLs, and test per RBP whether absolute deltaSVM differs between groups using a rank-based test with FDR correction at 0.05. Rationale: the paper's core finding is that while many variants affect splicing by altering splice-site sequences, many more modify RBP binding sites, so model-based allele-effect scoring captures a major sQTL mechanism.

## Related tools

- **gkmSVM** (R package for gapped-kmer SVM used to train per-RBP classifiers on eCLIP peak sequences and compute deltaSVM allele-effect scores)
- **sQTLseekeR2** (R package (run via the sqtlseeker2-nf Nextflow pipeline) producing the multivariate sQTL catalog whose variants are stratified for deltaSVM comparison) — https://github.com/guigolab/sqtlseeker2-nf
- **LeafCutter** (Annotation-free quantification of intron excision ratios providing the splicing phenotypes upstream of sQTL mapping)
- **Nextflow** (Workflow engine for the reproducible sqtlseeker2-nf sQTL mapping pipeline (with Docker)) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- rbp_deltaSVM_significance.tsv contains exactly 10 RBPs passing FDR < 0.05
- RBFOX2, PRPF8, SF3B4 and PPIG appear among the 10 significant RBPs
- |deltaSVM| is larger for sQTLs than non-sQTLs in 9 of the 10 significant proteins
- Only 79 RBP gkmSVM models are retained, each with ROC AUC >= 0.8
- Only RBPs with >= 30 top-5% sQTLs overlapping their eCLIP peaks enter the test (32 RBPs)

## Limitations

- Analysis is restricted to the 32 RBPs with at least 30 top-5% sQTLs overlapping high-confidence eCLIP peaks; RBPs below this overlap threshold are untested.
- Only gkmSVM models reaching ROC AUC >= 0.8 are retained, so poorly learned RBPs are excluded from deltaSVM comparison.
- sQTLs are defined by the multivariate sQTLseekeR2 framework (nominal plus permuted pass with FDR correction); variants in LD with the causal variant may confound attribution of deltaSVM differences.

## Evidence

- [abstract] While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins.: "While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins."
- [abstract] gkmSVM: an R package for gapped-kmer SVM: "gkmSVM: an R package for gapped-kmer SVM"
- [readme] sQTLseekeR2 is an enhanced version of sQTLseekeR, an R package to detect splicing QTLs (sQTLs).: [evidence span withheld — anchor status: external_doc]
- [readme] A pipeline for splicing quantitative trait loci (sQTL) mapping.: "A pipeline for splicing quantitative trait loci (sQTL) mapping."
