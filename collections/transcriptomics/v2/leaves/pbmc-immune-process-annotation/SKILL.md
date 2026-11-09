---
name: pbmc-immune-process-annotation
description: Use when a short list of EDGE feature genes (e.g., 35 genes from PBMC
  cells) needs biological-process interpretation, specifically testing whether they
  are enriched for immune- or stimulus-response-related GO terms.
when_to_use_negative:
- The gene list comes from a method other than EDGE feature-gene extraction or is
  not the μ − 1.5 × σ entropy-cutoff set — the enrichment result would not correspond
  to the EDGE-reported Table 3 terms
- You need whole-transcriptome or pathway-level (KEGG/Reactome) interpretation rather
  than GO biological-process over-representation of a small feature-gene set
- The dataset is not human PBMC scRNA-seq; the immune-response term expectations (defense
  response, leukocyte-mediated immunity) are PBMC-specific
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: EDGE
  role: ensemble dimensionality reduction (endr) with massive weak learners used to
    extract the 35 PBMC feature genes that feed the GO enrichment test
  repo: github.com/shawnstat/EDGE
- name: R
  role: implementation environment for the EDGE package, normalization (sweep/log2),
    and the GO over-representation workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:7437634d20206c69ce7a208b5ef280e746d0094514c5f67142cb7149d7ccb0cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EDGE
  - R
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pbmc-immune-process-annotation

## Summary

Annotates EDGE-detected feature genes from the PBMC scRNA-seq dataset by performing a GO biological-process over-representation test and producing a ranked table of the ten most enriched immune-response-related processes with fold enrichment, raw P-value, and FDR.

## When to use

Use when a short list of EDGE feature genes (e.g., 35 genes from PBMC cells) needs biological-process interpretation, specifically testing whether they are enriched for immune- or stimulus-response-related GO terms.

## When NOT to use

- The gene list comes from a method other than EDGE feature-gene extraction or is not the μ − 1.5 × σ entropy-cutoff set — the enrichment result would not correspond to the EDGE-reported Table 3 terms
- You need whole-transcriptome or pathway-level (KEGG/Reactome) interpretation rather than GO biological-process over-representation of a small feature-gene set
- The dataset is not human PBMC scRNA-seq; the immune-response term expectations (defense response, leukocyte-mediated immunity) are PBMC-specific

## Inputs

- PBMC scRNA-seq gene-by-cell UMI count matrix (from which log2-normalized cell-by-gene expression dat_log is built)
- List of feature genes selected by EDGE below the μ − 1.5 × σ averaged-entropy cutoff (35 genes for PBMC)
- Gene-universe background gene set and GO biological-process annotations (e.g., org.Hs.eg.db / clusterProfiler-style term mappings)

## Outputs

- table3_go_enrichment.csv: a ten-row table of enriched GO biological-process terms with columns GO term, fold enrichment, raw P-value, and FDR

## How to apply

First normalize the PBMC count matrix by library size to the median (sweep counts by rs/rs_med) and log2-transform as t(log2(dat_norm+1)) to feed the EDGE endr function with massive weak learners. Extract feature genes using the μ − 1.5 × σ cutoff on averaged entropy values, yielding ~35 PBMC feature genes. Run a hypergeometric over-representation test of these genes against all GO biological-process terms with an appropriate gene-universe background. Adjust raw P-values by FDR, compute fold enrichment per term, rank by significance, and keep the ten most enriched processes (expected to be immune response / response to stimulus terms such as defense response, leukocyte-mediated immunity, immune effector process).

## Related tools

- **EDGE** (ensemble dimensionality reduction (endr) with massive weak learners used to extract the 35 PBMC feature genes that feed the GO enrichment test) — github.com/shawnstat/EDGE
- **R** (implementation environment for the EDGE package, normalization (sweep/log2), and the GO over-representation workflow)

## Examples

```
custom_defs <- endr_defs; custom_defs$n_wl <- 1000; custom_defs$seed <- 2020; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- Output CSV contains exactly ten GO biological-process rows with fold enrichment, raw P-value, and FDR columns
- All ten enriched terms are immune/response-to-stimulus related (e.g., defense response, leukocyte-mediated immunity, immune effector process), matching Table 3
- The feature-gene input set has ~35 genes, i.e., those below the μ − 1.5 × σ cutoff of averaged entropy values
- FDR values are monotone-consistent with raw P-values after Benjamini–Hochberg-style adjustment and are < significance threshold for the retained rows
- Fold enrichment values are > 1 for every retained term, confirming over-representation rather than depletion

## Limitations

- The gene-universe background definition is not specified in the article, so fold enrichment and P-values are sensitive to that choice
- Only ten most-significant terms are retained; the full enrichment landscape of the 35 genes is not reported
- Large hash-table/weak-learner settings in EDGE require substantial memory; the README notes 'large memory will be required' and demonstrates a reduced hash table size of 101,107 instead of the optimal 1,017,881

## Evidence

- [discussion] Feature gene cutoff set at μ − 1.5 × σ of the averaged entropy values: "We choose μ −  1.5 × σ as the cutoff value to select top feature genes"
- [intro] EDGE ensemble uses massive weak learners that select feature genes: "massive weak learners are allowed in EDGE to learn cell similarities accurately, while the genes that make significant contributions in the process are selected as feature genes."
- [readme] Normalization and log2 transform recipe from the EDGE README: "dat_norm <- sweep(counts_dt,2,rs/rs_med,"/")
    dat_log <- t(log2(dat_norm+1))"
- [methods] endr function call for ensemble dimensionality reduction: "simu.endr <- endr(dat_log,custom_defs)"
