---
name: subclone-signature-overlap-analysis
description: Use when you have two gene lists — one from subclonal mutation-associated
  expression (scRNA-seq) and one from bulk expression correlation (e.g., TCGA AML,
  q < 0.001) — and need to test for a significant regulon-level overlap.
when_to_use_negative:
- The two gene lists are not independently derived (e.g., both from the same sample/cohort),
  which inflates overlap significance
- The input is a per-cell mutation-by-barcode matrix rather than a summarized gene
  list — derive subclone-associated genes first via expression association
- The subclone signature comes from copy number alterations rather than expressed
  SNV-defined subclones, since CNAs are rare in AML and rarely capture complete subclonal
  complexity
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: python3
  role: Runs the Pearson correlation, BH correction, and hypergeometric overlap test
    as a scripted analysis
- name: pysam
  role: Supports upstream read/barcode processing that produced the subclone-associated
    gene list being tested
- name: Docker
  role: Provides the reproducible execution environment (sridnona/python3:180925.v1)
    for the python3 analysis scripts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:89923fad77360fc6b08f71af5e9e8f2b0a08416bc8ccab83b2fa16e1acff1ddc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - pysam
  - python3
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# subclone-signature-overlap-analysis

## Summary

Tests whether a gene signature derived from a single-cell-defined mutation subclone (e.g., GATA2R361C subclone-associated genes) overlaps more than expected by chance with an independent bulk cohort signature (e.g., TCGA AML VIM-correlated genes) using a hypergeometric test, followed by GO enrichment of the intersecting genes.

## When to use

Use when you have two gene lists — one from subclonal mutation-associated expression (scRNA-seq) and one from bulk expression correlation (e.g., TCGA AML, q < 0.001) — and need to test for a significant regulon-level overlap.

## When NOT to use

- The two gene lists are not independently derived (e.g., both from the same sample/cohort), which inflates overlap significance
- The input is a per-cell mutation-by-barcode matrix rather than a summarized gene list — derive subclone-associated genes first via expression association
- The subclone signature comes from copy number alterations rather than expressed SNV-defined subclones, since CNAs are rare in AML and rarely capture complete subclonal complexity

## Inputs

- TCGA AML expression matrix (bulk, with sample-level gene expression including VIM)
- GATA2R361C subclone-associated gene list (produced by task_006 from single-cell analysis)

## Outputs

- VIM-correlated gene set (2191 genes, q < 0.001 after Benjamini-Hochberg correction)
- Hypergeometric overlap p-value (p = 3.5e-95) and intersection of ~198 genes
- ToppFun GO enrichment terms (immune response, cytoskeletal organization, focal adhesion)
- vim_gata2_overlap_results.tsv (intersection gene list, p-value, GO terms)

## How to apply

First derive both gene sets: compute Pearson correlation of every gene with the anchor gene (VIM) across TCGA AML samples, Benjamini-Hochberg correct the p-values, and retain genes with q < 0.001 (2191 genes); take the subclone-associated gene list from the prior single-cell genotyping/expression association step (task_006 output for GATA2R361C). Then run a scipy.stats hypergeometric test in a python3 (Docker sridnona/python3:180925.v1) environment to assess whether the intersection exceeds chance — here ~198 shared genes, p = 3.5e-95 — which supports a VIM regulon linked to the GATA2R361C subclone. Finally, submit the intersecting genes to ToppFun for GO enrichment covering immune response, cytoskeletal organization, and focal adhesion, and write the intersection list, p-value, and GO terms to vim_gata2_overlap_results.tsv.

## Related tools

- **python3** (Runs the Pearson correlation, BH correction, and hypergeometric overlap test as a scripted analysis)
- **pysam** (Supports upstream read/barcode processing that produced the subclone-associated gene list being tested)
- **Docker** (Provides the reproducible execution environment (sridnona/python3:180925.v1) for the python3 analysis scripts)

## Examples

```
python3 overlap_hypergeom.py --set1 vim_corr_q001.txt --set2 gata2r361c_subclone.txt --background all_genes.txt  # scipy.stats.hypergeom; expect ~198 genes, p=3.5e-95
```

## Evaluation signals

- Intersection of TCGA VIM-correlated genes (q < 0.001) with GATA2R361C subclone genes contains ~198 genes
- Hypergeometric overlap test reproduces p = 3.5e-95
- VIM-correlated set size matches the expected 2191 genes at q < 0.001
- ToppFun recovers GO terms for immune response, cytoskeletal organization, and focal adhesion
- vim_gata2_overlap_results.tsv is written with intersection gene list, p-value, and GO terms

## Limitations

- Dropout (transcript and allelic) and partial transcript coverage on end-biased platforms limit sensitivity of the mutation detection underlying the subclone signatures
- It is impossible to determine whether a cell is truly wild-type for a given mutation, so subclone membership is probabilistic
- Overlap significance depends on the specific mutational composition of the sample; the approach may perform better in tumor types with higher mutation burdens than AML

## Evidence

- [other] The overlap between TCGA VIM-correlated genes and GATA2R361C subclone-associated genes is statistically significant (p = 3.5 × 10–95 by hypergeometric test): [evidence span withheld — anchor status: not_found]
- [abstract] identify expression signatures associated with subclonal mutations, and find cell surface markers that could be used to purify subclones: "identify expression signatures associated with subclonal mutations, and find cell surface markers that could be used to purify subclones"
- [other] Use scipy.stats hypergeometric testing (via a python3 script) to assess whether the VIM-correlated set overlaps the GATA2R361C subclone gene list: [evidence span withheld — anchor status: not_found]
- [discussion] CNAs rarely reflect the complete clonal architecture of a tumor (and are rare in most AML samples): "Because CNAs rarely reflect the complete clonal architecture of a tumor (and are rare in most AML samples)"
- [readme] Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data"
