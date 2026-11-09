---
name: statistical-link-concordance-comparison
description: Use when you need to test whether regression-based (linear + interaction
  term) differential gene–gene links are also recoverable by a non-linear dependence
  measure such as k-NN mutual information, on matched subgroup data.
when_to_use_negative:
- The goal is only DEGGs' own linear-model output (regression with interaction term)
  and no non-linear comparison is wanted — the MI step adds no value
- Expression data has only one phenotype group or no subgroup_variable to stratify,
  so differential co-expression between subgroups cannot be defined
- Input data are not normalised counts compatible with generate_subnetworks' expected
  count/design matrices
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3360
tools:
- name: DEGGs
  role: 'Provides the linear differential co-expression calls being compared: generate_subnetworks
    fits robust linear regressions with an interaction term per gene–gene link, and
    extract_sig_deggs lists the significant pairs'
  repo: https://github.com/elisabettasciacca/DEGGs
- name: R
  role: Implementation environment for loading BRCA_normCounts/BRCA_metadata, running
    DEGGs functions, and computing k-NN mutual information estimates with permutation
    thresholds
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad192
    title: 'DEGGs: an R package with shiny app for the identification of differentially
      expressed gene–gene interactions in high-throughput sequencing data'
  source_content_hash: sha256:d88ab262721ab8cd3f8ee0f63b8ccb451b441a29b8897dc2de227640281fb48f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elisabettasciacca/DEGGs
  tools:
  - DEGGs
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# statistical-link-concordance-comparison

## Summary

Benchmarks DEGGs' linear-model differential co-expression calls against a non-linear association test (mutual information) on the same gene–gene links from the TCGA-BRCA subnetworks, quantifying which links each statistical framework detects.

## When to use

Use when you need to test whether regression-based (linear + interaction term) differential gene–gene links are also recoverable by a non-linear dependence measure such as k-NN mutual information, on matched subgroup data.

## When NOT to use

- The goal is only DEGGs' own linear-model output (regression with interaction term) and no non-linear comparison is wanted — the MI step adds no value
- Expression data has only one phenotype group or no subgroup_variable to stratify, so differential co-expression between subgroups cannot be defined
- Input data are not normalised counts compatible with generate_subnetworks' expected count/design matrices

## Inputs

- BRCA_normCounts normalised count matrix (packaged TCGA-BRCA RNA-seq data)
- BRCA_metadata clinical metadata with a SUBTYPE column (e.g. BRCA_Her2, BRCA_LumA)
- DEGGs subnetworks object produced by generate_subnetworks
- Significant gene–gene pair table from extract_sig_deggs

## Outputs

- nonlinear_vs_linear_links_comparison.csv with one row per DEGGs-significant pair, columns gene_A, gene_B, linear p/q-value, MI value, detected_by_linear, detected_by_MI
- Per-link record pairing the linear-model p/q-value with the mutual information statistic

## How to apply

Load the packaged TCGA-BRCA data with data("BRCA_normCounts") and data("BRCA_metadata"), verifying column/row alignment with identical(). Run DEGGs::generate_subnetworks on the normalised counts with subgroup_variable = 'SUBTYPE' and subgroups = c('BRCA_Her2', 'BRCA_LumA'), entrezIDs = TRUE and convert_to_gene_symbols = TRUE, keeping the default robust linear regression with an interaction term. Extract the significant gene–gene pairs with DEGGs::extract_sig_deggs to get the FDR-filtered linear-model links. For every extracted pair, compute a mutual information estimate between the two genes' expression vectors within each subgroup using a k-nearest-neighbour MI estimator, and declare the link detected by the non-linear test when MI exceeds a permutation-derived significance threshold matched to the linear test's FDR-controlled calls. Write the combined per-link comparison table (gene_A, gene_B, linear p/q-value, MI value, detected_by_linear, detected_by_MI) to nonlinear_vs_linear_links_comparison.csv, retaining the linear p/q-value alongside the MI statistic for each pair.

## Related tools

- **DEGGs** (Provides the linear differential co-expression calls being compared: generate_subnetworks fits robust linear regressions with an interaction term per gene–gene link, and extract_sig_deggs lists the significant pairs) — https://github.com/elisabettasciacca/DEGGs
- **R** (Implementation environment for loading BRCA_normCounts/BRCA_metadata, running DEGGs functions, and computing k-NN mutual information estimates with permutation thresholds)

## Examples

```
subnetworks_object <- generate_subnetworks(BRCA_normCounts, BRCA_metadata, subgroup_variable = "SUBTYPE", subgroups = c("BRCA_Her2", "BRCA_LumA"), entrezIDs = TRUE, cores = 2); sig_pairs <- extract_sig_deggs(subnetworks_object)
```

## Evaluation signals

- The comparison table contains exactly one row per DEGGs-significant pair extracted by extract_sig_deggs
- Every row carries both a linear-model significance value (p and q) and an MI value — no missing fields
- Pairs concordantly detected by both the linear test and the MI test form a majority subset of the linear-model links
- MI significance thresholds are permutation-derived and matched to the linear test's FDR-controlled calls rather than arbitrary cutoffs
- gene–gene pair identities in the CSV match the extract_sig_deggs output (gene_A/gene_B naming consistent, e.g. NOTCH2–DTX4 style pairs)

## Limitations

- DEGGs cannot detect non-linear gene–gene relations; the authors explicitly state this as a package limitation since it relies on linear regressions with an interaction term to assess differential co-expression
- The DEGGs repository is no longer maintained — the package has been moved and significantly expanded into multiDEGGs
- MI-based detection depends on the permutation-derived threshold calibration; a mismatch between MI thresholds and the linear test's FDR-controlled calls can inflate discordance

## Evidence

- [abstract] Linear-model significance per link: "For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term"
- [methods] Subgroup differential correlations: "The DEGGs package enables the exploration of differential gene-gene correlations in different phenotipical groups."
- [methods] Significant pair extraction: "To get a table listing all the significant gene-gene pairs found in all subgroups, you will need to use the extract_sig_deggs"
- [readme] Data loading and network generation: [evidence span withheld — anchor status: external_doc]
