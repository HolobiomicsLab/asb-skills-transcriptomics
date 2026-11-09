---
name: fdr-multiple-testing-correction
description: Use when many statistical tests (e.g., Fisher's exact tests of resource-annotation
  enrichment across SignaLink, NetPath, CancerSEA terms) are run simultaneously and
  per-test p-values must be corrected before calling over-/underrepresentation.
when_to_use_negative:
- Only a single hypothesis test is run per resource-annotation pair — no family of
  tests exists to correct over
- The comparison is between CCC inference methods (expression-based scores) rather
  than between resource interaction contents and a pooled background
- Effect direction and magnitude (odds ratios) are not needed and raw uncorrected
  exploratory p-values suffice
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: LIANA
  role: Provides the 16 CCC resources whose ligand-receptor interactions are compared
    against the pooled background, generating the per-resource p-value families that
    are BH-corrected
  repo: https://github.com/saezlab/liana
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:4de058e5addd9956ecc09644f996c1ea212f0834aefd6d9574f31622dc7cdafa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - LIANA
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# fdr-multiple-testing-correction

## Summary

Applies Benjamini-Hochberg false-discovery-rate correction to per-resource Fisher's exact test p-values computed over many annotation terms (pathways, cancer gene sets, tissue-enriched proteins), so that resource-vs-background enrichment calls in cell-cell communication resources are made at a controlled false-positive rate.

## When to use

Use when many statistical tests (e.g., Fisher's exact tests of resource-annotation enrichment across SignaLink, NetPath, CancerSEA terms) are run simultaneously and per-test p-values must be corrected before calling over-/underrepresentation.

## When NOT to use

- Only a single hypothesis test is run per resource-annotation pair — no family of tests exists to correct over
- The comparison is between CCC inference methods (expression-based scores) rather than between resource interaction contents and a pooled background
- Effect direction and magnitude (odds ratios) are not needed and raw uncorrected exploratory p-values suffice

## Inputs

- Vector of raw Fisher's exact test p-values, one per resource-annotation pair (annotation terms from SignaLink, NetPath, CancerSEA, Human Protein Atlas, DisGeNet)
- Per-resource 2x2 contingency tables built from ligand-receptor interaction lists of the 16 CCC resources in LIANA
- Pooled 16-resource background collection of ligand-receptor interactions

## Outputs

- Vector of BH-adjusted (FDR-corrected) p-values per resource-annotation pair
- Significance calls (FDR-significant over-/underrepresentation) per resource-annotation pair
- Odds-ratio matrix annotated with significance, used for the figure4_odds_ratio_heatmap.png heatmap

## How to apply

For each of the 16 CCC resources assembled in LIANA, compute Fisher's exact test p-values from 2x2 contingency tables (in-annotation vs out-of-annotation, resource vs the pooled 16-resource background) for every annotation term. Collect the per-resource vector of raw p-values across annotation terms, then apply Benjamini-Hochberg FDR correction (R's p.adjust with method='BH') within each resource. Corrected p-values below the significance threshold flag annotation terms whose coverage differs from the pooled background — e.g., the T cell receptor pathway underrepresented in Ramilowski, ConnectomeDB, CellTalkDB, LRdb, and iTALK but overrepresented in OmniPath and Cellinker. Pair the corrected p-values with odds ratios to build the odds-ratio matrix used for the Fig 4 heatmap, marking only FDR-significant cells.

## Related tools

- **LIANA** (Provides the 16 CCC resources whose ligand-receptor interactions are compared against the pooled background, generating the per-resource p-value families that are BH-corrected) — https://github.com/saezlab/liana

## Examples

```
pvals_adj <- p.adjust(fisher_pvals_per_resource, method = "BH")  # per-resource vector of Fisher's exact p-values
```

## Evaluation signals

- Adjusted p-values are monotonically non-decreasing relative to raw p-values (BH invariant), and no adjusted p-value is smaller than its raw counterpart
- Each resource's p-value family is corrected separately across annotation terms (per-resource correction), not pooled across resources
- FDR-significant cells in the odds-ratio heatmap reproduce the paper's calls: T cell receptor pathway underrepresented in Ramilowski, ConnectomeDB, CellTalkDB, LRdb, iTALK and overrepresented in OmniPath and Cellinker
- Significance annotations on the heatmap correspond exactly to adjusted p-values passing the FDR threshold, not to raw p-values

## Limitations

- FDR correction controls the expected false-discovery proportion across annotation terms per resource but does not establish biological ground truth — the article notes 'no gold standard is currently available and the biological ground truth for CCC is largely unknown'
- Enrichment results depend on the pooled 16-resource background definition; changing the set of resources changes the contingency tables and therefore the corrected significance calls
- Uneven pathway and tissue-enriched protein coverage means sparse resources may have unstable Fisher's exact tests for small annotation sets

## Evidence

- [methods] Contingency table and Fisher's exact test per resource: [evidence span withheld — anchor status: not_found]
- [methods] BH FDR correction of per-resource p-values: [evidence span withheld — anchor status: not_found]
- [abstract] Motivating finding: uneven coverage of pathways and tissue-enriched proteins: "Among the resources, we find few unique interactions, a varying degree of overlap, and an uneven coverage of specific pathways and tissue-enriched proteins."
- [results] T cell receptor pathway over-/underrepresentation calls: [evidence span withheld — anchor status: not_found]
