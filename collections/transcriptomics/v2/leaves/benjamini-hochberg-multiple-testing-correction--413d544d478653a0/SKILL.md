---
name: benjamini-hochberg-multiple-testing-correction
description: Use when many paired statistical tests (e.g., matched perturbed-vs-control
  Wilcoxon tests over hundreds of TF-gene edges) have been run and raw p-values must
  be FDR-adjusted before calling significant edges.
when_to_use_negative:
- Only a single hypothesis test is being run per comparison (no multiplicity, no correction
  needed)
- The goal is per-TF family-wise error control rather than FDR across the full edge
  set
- The expression data are not matched perturbed/control samples from the same fixed
  batch, so paired tests are inappropriate
edam_operation: http://edamontology.org/operation_0579
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GRouNdGAN
  role: Generates paired perturbed/unperturbed single-cell samples per TF knockout
    via --perturb, supplying the p-values that are then FDR-corrected
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Provides the imposed TF-gene edge list whose per-edge significance fractions
    are computed after correction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:6df5946fe8d11c508964e61798c6855875eb71799c01e4862aede308234e0787
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - GRNBoost2
  - GRouNdGAN
  edam_operation: http://edamontology.org/operation_0579
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Applies Benjamini-Hochberg false-discovery-rate correction to Wilcoxon test p-values across all TF-gene edges from GRouNdGAN in silico TF knockout experiments, so edge significance (adjusted p < 0.05) can be counted to quantify how much of an imposed GRN is causally enforced in simulated cells.

## When to use

Use when many paired statistical tests (e.g., matched perturbed-vs-control Wilcoxon tests over hundreds of TF-gene edges) have been run and raw p-values must be FDR-adjusted before calling significant edges.

## When NOT to use

- Only a single hypothesis test is being run per comparison (no multiplicity, no correction needed)
- The goal is per-TF family-wise error control rather than FDR across the full edge set
- The expression data are not matched perturbed/control samples from the same fixed batch, so paired tests are inappropriate

## Inputs

- Raw p-values from two-sided Wilcoxon signed-rank tests for each TF-gene edge in the imposed GRN
- Matched perturbed vs. control expression matrices sampled from the same fixed cell batch by GRouNdGAN --perturb
- Imposed GRN edge list (TF-gene pairs) from GRNBoost2 or PPCOR

## Outputs

- Benjamini-Hochberg adjusted p-values for all TF-gene edges
- Set of significant edges (adjusted p < 0.05)
- knockout_edge_significance.csv with per-edge test results
- Fraction of significantly perturbed TF-gene edges per imposed GRN (e.g., 66.5% for GRNBoost2, 75.2% for PPCOR)

## How to apply

After each in silico TF knockout in GRouNdGAN (using the --perturb command with [Perturbation]/perturbation set to 0 and matched perturbed/control cells sampled from the same fixed batch), run a two-sided Wilcoxon signed-rank test on the expression of each gene the knocked-out TF regulates in the imposed GRN. Pool the raw p-values from all TF-gene edges and apply the Benjamini-Hochberg FDR procedure across the full edge set. Call an edge significant at adjusted p-value < 0.05, then compute the fraction of significantly perturbed edges separately for the GRNBoost2 GRN and the PPCOR GRN. The rationale is that thousands of edges are tested simultaneously, so uncorrected p-values would massively inflate the apparent knockout effect; correction yields interpretable per-edge significance suitable for reproducing the paper's 66.5% (GRNBoost2) and 75.2% (PPCOR) rates.

## Related tools

- **GRouNdGAN** (Generates paired perturbed/unperturbed single-cell samples per TF knockout via --perturb, supplying the p-values that are then FDR-corrected) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Provides the imposed TF-gene edge list whose per-edge significance fractions are computed after correction)

## Examples

```
from statsmodels.stats.multitest import multipletests; rejected, padj, _, _ = multipletests(wilcoxon_pvals, alpha=0.05, method='fdr_bh')
```

## Evaluation signals

- The fraction of significant (adjusted p < 0.05) edges reproduces ~66.5% for the GRNBoost2 GRN within small numerical tolerance
- The fraction of significant edges reproduces ~75.2% for the PPCOR GRN within small numerical tolerance
- knockout_edge_significance.csv contains one row per TF-gene edge with both raw and adjusted p-values
- The number of corrected p-values equals the number of edges in the imposed GRN (no edges dropped or duplicated)
- Unperturbed control sampling used the same fixed batch as perturbed sampling, ensuring matched Wilcoxon pairs

## Limitations

- GRouNdGAN only supports bipartite GRNs (single-layer TF-gene edges), so correction applies to that edge set only; multi-layer causal graphs are not supported
- The model is generative, not predictive, and must be retrained per dataset; corrected significance rates are specific to the trained checkpoint and imposed GRN
- Resemblance of simulated data to the reference deteriorates if imposed TF-gene relationships differ significantly from reference data patterns, weakening knockout effect sizes

## Evidence

- [methods] Wilcoxon p-values with Benjamini-Hochberg FDR correction: [evidence span withheld — anchor status: not_found]
- [results] 66.5% significant knockout edges: [evidence span withheld — anchor status: not_found]
- [methods] perturbation command: "--perturb        Perform a perturbation experiment using a trained GRouNdGAN model"
- [methods] two-sided Wilcoxon test: [evidence span withheld — anchor status: not_found]
