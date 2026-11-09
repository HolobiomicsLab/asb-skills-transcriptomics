---
name: nmf-clustering-of-rbp-feature-matrices
description: Use when a merged candidate-RBP feature matrix (e.g., MATRIX.txt) must
  be partitioned into clusters whose enrichment for known circRNA regulators (gold_regulators.xlsx)
  is tested to select candidate regulators.
when_to_use_negative:
- Input is already a cluster-assignment file (NMFgroup_*.txt) or a single gene-expression
  matrix rather than the five-feature candidate-regulator matrix
- The goal is de novo identification of circRNA-associated genes rather than clustering
  of a pre-defined candidate RBP list
- Protein-RNA interaction data has not been filtered by RNAct interaction score ≥
  30 and the C value feature has not been computed, so the feature matrix is incomplete
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CRIT
  role: Pipeline providing the feature matrix, NMF.R clustering script, RBP_list.RData,
    and Fisher's exact test validation step (cluster identify + Validate candidates
    components)
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: R
  role: Execution environment; NMF package performs clustering, R stats package performs
    Fisher's exact enrichment tests, survival package implements the underlying log-rank
    Zs feature
- name: NMF
  role: R package that applies the non-negative matrix factorization to the features
    matrix with cluster number selected by CPCC and dispersion indices
  repo: https://cran.r-project.org/web/packages/NMF/NMF.pdf
- name: NMF.R
  role: Repository script ('NMF method to cluster candidate RBPs') that runs the factorization
    and writes NMFgroup_XXclusterYYrun.txt outputs
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: RBP_analysis.R
  role: Repository script for candidate RBPs validation, i.e., the Fisher's exact
    enrichment tests against gold_regulators.xlsx
  repo: https://github.com/BioinformaticsSTU/CRIT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:60b96861837bf94b61dc9d6947b0d13ea3dbf6c3bef842c645e2e34308a2fad5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  repo_url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CRIT
  - NMF
  - NMF.R
  - R
  - RBP_analysis.R
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# nmf-clustering-of-rbp-feature-matrices

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters candidate RNA-binding proteins (RBPs) by applying non-negative matrix factorization (NMF) to the CRIT multi-omics feature matrix (ρm, Zp, Zs, Zg, C value), then identifies the gold-regulator-enriched cluster via Fisher's exact tests. This is the 'cluster identify' step of the CRIT pipeline for circRNA regulator discovery.

## When to use

Use when a merged candidate-RBP feature matrix (e.g., MATRIX.txt) must be partitioned into clusters whose enrichment for known circRNA regulators (gold_regulators.xlsx) is tested to select candidate regulators.

## When NOT to use

- Input is already a cluster-assignment file (NMFgroup_*.txt) or a single gene-expression matrix rather than the five-feature candidate-regulator matrix
- The goal is de novo identification of circRNA-associated genes rather than clustering of a pre-defined candidate RBP list
- Protein-RNA interaction data has not been filtered by RNAct interaction score ≥ 30 and the C value feature has not been computed, so the feature matrix is incomplete

## Inputs

- MATRIX.txt — merged candidate regulator feature matrix (features ρm, Zp, Zs, Zg, C value)
- RBP_list.RData — list of 1344 candidate RBP genes
- gold_regulators.xlsx — list of gold-standard circRNA regulators from previous research
- Optional validation lists: Li et al. 103-RBP circmCherry screen hits; 19 curated established regulators

## Outputs

- NMFgroup_XXclusterYYrun.txt — cluster assignment per gene for each number of clusters (XX) and run (YY)
- Fisher's exact test p-values for enrichment of gold regulators in each cluster (reference: cluster 23, 90 RBPs, 17 gold regulators, p = 0.00012)

## How to apply

Load the merged candidate RBP feature matrix (MATRIX.txt) produced by GET_feauters_MATRIX.R and the list of 1344 candidate RBPs from RBP_list.RData. Run the NMF algorithm (R 'NMF' package, script NMF.R) with the number of clusters R=36, chosen from local maxima of the cophenetic correlation coefficient (CPCC) and dispersion (DC) indices, and write each run's assignments to NMFgroup_36clusterYYrun.txt using the XXclusterYYrun naming convention. Then perform Fisher's exact tests in R comparing each cluster's membership against gold_regulators.xlsx to find the significantly enriched cluster — in the reference implementation this is cluster 23 with 90 RBPs of which 17 are gold-standard circRNA regulators (p = 0.00012). Reproduce the supporting overlap tests: 13/90 against Li et al.'s 103-RBP circmCherry screen (p = 0.00201) and 6/19 against curated established regulators (p = 0.00112). The rationale is that proteins sharing the five CRIT features (expression correlation with circRNAs, alteration proportion, survival impact, GO:0000398 spliceosome similarity, circRNA-binding C value) group together, so a cluster enriched in known regulators captures the regulatory phenotype.

## Related tools

- **CRIT** (Pipeline providing the feature matrix, NMF.R clustering script, RBP_list.RData, and Fisher's exact test validation step (cluster identify + Validate candidates components)) — https://github.com/BioinformaticsSTU/CRIT
- **R** (Execution environment; NMF package performs clustering, R stats package performs Fisher's exact enrichment tests, survival package implements the underlying log-rank Zs feature)
- **NMF** (R package that applies the non-negative matrix factorization to the features matrix with cluster number selected by CPCC and dispersion indices) — https://cran.r-project.org/web/packages/NMF/NMF.pdf
- **NMF.R** (Repository script ('NMF method to cluster candidate RBPs') that runs the factorization and writes NMFgroup_XXclusterYYrun.txt outputs) — https://github.com/BioinformaticsSTU/CRIT
- **RBP_analysis.R** (Repository script for candidate RBPs validation, i.e., the Fisher's exact enrichment tests against gold_regulators.xlsx) — https://github.com/BioinformaticsSTU/CRIT

## Examples

```
Rscript NMF.R  # NMF method to cluster candidate RBPs from MATRIX.txt, then Rscript RBP_analysis.R  # Fisher's exact enrichment tests against gold_regulators.xlsx
```

## Evaluation signals

- The gold-regulator-enriched cluster contains exactly 90 RBPs, of which 17 are gold-standard circRNA regulators, with Fisher's exact test p = 0.00012
- Overlap with Li et al.'s 103-RBP circmCherry screen is 13/90 with p = 0.00201
- Overlap with the 19 curated established regulators is 6/19 with p = 0.00112
- Cluster number R = 36 corresponds to local maxima of the cophenetic correlation coefficient (CPCC) and dispersion (DC) indices
- Output files follow the XXclusterYYrun naming convention (e.g., NMFgroup_36clusterYYrun.txt) with one assignment per candidate RBP from RBP_list.RData

## Limitations

- Cluster results depend on the run count and cluster number; reproducibility requires matching the reference settings (R = 36, multiple runs per configuration)
- The pipeline's enrichment logic is tied to the gold_regulators.xlsx list collected from previous research; different gold-standard lists may shift which cluster is enriched
- Generalization of the CRIT strategy beyond cancers (e.g., to neurodegenerative disease) is untested per the discussion
- The candidate space is limited to pre-defined RBP lists; supervised-learning-based prediction of circRNA-regulating RBPs remains an unimplemented extension

## Evidence

- [results] NMF clustering yielded cluster 23 containing 90 RBPs, of which 17 are gold-standard circRNA regulators, a significant enrichment (Fisher's exact test p = 0.00012): [evidence span withheld — anchor status: not_found]
- [methods] Run the NMF algorithm (R 'NMF' package, NMF.R) on the features matrix with rank R=36 selected by local maxima of the cophenetic correlation coefficient (CPCC) and dispersion (DC) indices.: [evidence span withheld — anchor status: not_found]
