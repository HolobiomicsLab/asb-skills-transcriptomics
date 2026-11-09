---
name: robust-regression-interaction-testing
description: Use when testing whether a gene-gene correlation differs between phenotypic
  subgroups (e.g. BRCA_Her2 vs BRCA_LumA) in normalized RNA-seq count data, and per-link
  differential p-values are needed.
when_to_use_negative:
- When comparing more than two groups with the robust-regression path — switch to
  the one-way ANOVA variant instead
- When the input is raw (unnormalized) count data or multi-omic assays — the successor
  multiDEGGs package with get_diffNetworks is the maintained route
- When metadata and count matrix columns/rows are not sample-aligned (identical(colnames,
  rownames) fails) — correct alignment first
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: DEGGs
  role: Implements per-link robust linear regression with interaction term (calc_pvalues_network
    / calc_pvalues_percentile) inside generate_subnetworks; results exposed via extract_sig_deggs
  repo: https://github.com/elisabettasciacca/DEGGs
- name: MASS
  role: Supplies the robust regression fitting (e.g. MASS::rlm) used for GeneA ~ GeneB
    * Group models
- name: qvalue
  role: Provides q-value-based multiple-testing correction via the use_qvalues option
- name: R
  role: Runtime environment in which the regression testing and p-value assembly are
    executed
  repo: https://github.com/elisabettasciacca/DEGGs
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_005
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad192
    title: 'DEGGs: an R package with shiny app for the identification of differentially
      expressed gene–gene interactions in high-throughput sequencing data'
  source_content_hash: sha256:6b6ff9d9abd31b5bdc44d1cf2185713c19a8cf6cbe11361e25472fe953ed8816
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elisabettasciacca/DEGGs
  tools:
  - DEGGs
  - MASS
  - R
  - org.Hs.eg.db
  - qvalue
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# robust-regression-interaction-testing

## Summary

Assess the differential significance of each gene-gene link across sample subgroups by fitting robust linear regression with a Gene*Group interaction term (one-way ANOVA when >2 groups) and extracting the F-test p-value on the interaction, as implemented in the DEGGs package.

## When to use

Use when testing whether a gene-gene correlation differs between phenotypic subgroups (e.g. BRCA_Her2 vs BRCA_LumA) in normalized RNA-seq count data, and per-link differential p-values are needed.

## When NOT to use

- When comparing more than two groups with the robust-regression path — switch to the one-way ANOVA variant instead
- When the input is raw (unnormalized) count data or multi-omic assays — the successor multiDEGGs package with get_diffNetworks is the maintained route
- When metadata and count matrix columns/rows are not sample-aligned (identical(colnames, rownames) fails) — correct alignment first

## Inputs

- BRCA_normCounts normalized count matrix (TCGA-BRCA RNA-seq)
- BRCA_metadata clinical table aligned by sample ID (rownames must match colnames of count matrix)
- subgroup_variable and subgroups specification, e.g. SUBTYPE with c('BRCA_Her2','BRCA_LumA')
- regression_method choice: robust linear regression (2 groups) or one-way ANOVA (>2 groups)

## Outputs

- deggs_pvalues.csv with one row per tested gene-gene pair and gene_A, gene_B, p-value and significance-flag columns
- Pair-level F-test p-values on the GeneB:Group interaction term for every gene-gene link
- Significant differentially interacting gene-gene pairs recoverable via extract_sig_deggs(subnetworks_object)

## How to apply

Load the packaged BRCA_normCounts normalized count matrix and BRCA_metadata clinical table, verifying identical(colnames(BRCA_normCounts), rownames(BRCA_metadata)), and subset metadata to the target subgroups via the subgroup_variable (e.g. 'SUBTYPE' with 'BRCA_Her2' and 'BRCA_LumA'). For each gene-gene pair, fit the robust linear model GeneA ~ GeneB * Group (e.g. MASS::rlm via calc_pvalues_network / calc_pvalues_percentile); if more than two groups are compared, switch to one-way ANOVA. Extract the F-test p-value of the GeneB:Group interaction term for each pair, which quantifies how much the gene-gene relationship differs across groups. Assemble all pair-level p-values into a table and flag pairs significant after multiple-testing correction, controlled by use_qvalues (q-value-based correction). Use these p-values to label links as group-specific within the DEGGs subnetworks object.

## Related tools

- **DEGGs** (Implements per-link robust linear regression with interaction term (calc_pvalues_network / calc_pvalues_percentile) inside generate_subnetworks; results exposed via extract_sig_deggs) — https://github.com/elisabettasciacca/DEGGs
- **MASS** (Supplies the robust regression fitting (e.g. MASS::rlm) used for GeneA ~ GeneB * Group models)
- **qvalue** (Provides q-value-based multiple-testing correction via the use_qvalues option)
- **R** (Runtime environment in which the regression testing and p-value assembly are executed) — https://github.com/elisabettasciacca/DEGGs

## Examples

```
subnetworks_object <- generate_subnetworks(normalised_counts = BRCA_normCounts, metadata = BRCA_metadata, subgroup_variable = "SUBTYPE", subgroups = c("BRCA_Her2", "BRCA_LumA"), entrezIDs = TRUE, convert_to_gene_symbols = TRUE, cores = 2)
```

## Evaluation signals

- deggs_pvalues.csv exists and contains one row per tested gene-gene pair with gene_A, gene_B, p-value and significance-flag columns
- Significant rows in deggs_pvalues.csv agree with the pairs listed by extract_sig_deggs(subnetworks_object)
- Every reported significant link is explainable by a low F-test p-value on its GeneB:Group interaction term
- colnames(BRCA_normCounts) is identical to rownames(BRCA_metadata) before fitting, so no sample mismatch distorts p-values
- Known HER2-positive-specific links KDR-FGF17 and SHC3-PDGFC appear among the significant interactions in the BRCA_Her2 vs BRCA_LumA run

## Limitations

- The original DEGGs repository is no longer maintained; the package has been moved and significantly expanded into multiDEGGs, which should be preferred for new work
- Robust linear regression with an interaction term is only appropriate for two-group comparisons; for >2 groups the method reverts to one-way ANOVA F-tests
- Per-pair p-values require multiple-testing adjustment (use_qvalues for q-value correction) to control false positives across all tested gene-gene links

## Evidence

- [abstract] Per-link regression method: "For each gene–gene link, differential statistical significance is provided through robust linear regression with an interaction term"
- [methods] Model specification and group handling: [evidence span withheld — anchor status: not_found]
- [readme] Data used for testing: [evidence span withheld — anchor status: external_doc]
- [abstract] Group-specific networks from count + design inputs: "The presented R package, provided with a count and design matrix, extract networks of group-specific interactions"
