---
name: wilcoxon-signed-rank-testing
description: Use when you have paired perturbed/unperturbed expression samples from
  matched cells (e.g., GRouNdGAN knockout vs control) and need per-edge significance
  of TF-gene regulation with multiple-testing correction.
when_to_use_negative:
- You want to predict whether TF knockout will change expression in real, non-simulated
  cells — GRouNdGAN is generative, not predictive, and would need retraining and a
  high-quality GRN plus data modalities such as scATAC-seq or TF ChIP-seq
- Perturbed and control cells are not matched (independently sampled), since the signed-rank
  test assumes paired differences
- The GRN contains multi-layer TF-TF-gene structure — the current GRouNdGAN version
  only supports bipartite GRNs
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GRouNdGAN
  role: Generates matched perturbed (TF knockout, value 0) and unperturbed cells from
    a trained checkpoint via the --perturb command, providing the paired expression
    samples tested with the Wilcoxon signed-rank test
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:10908348a7265b82f957347e8f7528788c5811c81d1273a787e5fe5d14e4fed9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - GRouNdGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Wilcoxon signed-rank testing

## Summary

Quantifies whether in silico TF knockout in GRouNdGAN-simulated cells significantly alters target-gene expression by applying two-sided Wilcoxon signed-rank tests to matched perturbed-vs-control expression, with Benjamini-Hochberg FDR correction over TF-gene edges.

## When to use

Use when you have paired perturbed/unperturbed expression samples from matched cells (e.g., GRouNdGAN knockout vs control) and need per-edge significance of TF-gene regulation with multiple-testing correction.

## When NOT to use

- You want to predict whether TF knockout will change expression in real, non-simulated cells — GRouNdGAN is generative, not predictive, and would need retraining and a high-quality GRN plus data modalities such as scATAC-seq or TF ChIP-seq
- Perturbed and control cells are not matched (independently sampled), since the signed-rank test assumes paired differences
- The GRN contains multi-layer TF-TF-gene structure — the current GRouNdGAN version only supports bipartite GRNs

## Inputs

- Matched perturbed and unperturbed expression matrices sampled from a trained GRouNdGAN model (.pth checkpoint, e.g. step_1000000.pth) via --perturb with perturbation value 0
- The imposed GRN (GRNBoost2- or PPCOR-derived bipartite TF-gene causal graph) defining which genes each knocked-out TF regulates

## Outputs

- Per-edge significance table knockout_edge_significance.csv (TF-gene edge, Wilcoxon p-value, BH-adjusted p-value)
- Fraction of significantly perturbed TF-gene edges per GRN (e.g., 66.5% for GRNBoost2, 75.2% for PPCOR on PBMC-CTL)

## How to apply

For each TF one at a time, sample perturbed and unperturbed cells from the same fixed batch (deterministic causal-controller mode, perturbation value set to 0) so the case-control comparison is matched. For each gene the TF regulates in the imposed GRN (e.g., GRNBoost2- or PPCOR-imposed), apply a two-sided Wilcoxon signed-rank test comparing matched perturbed vs. control expression across cells. Adjust all resulting p-values across TF-gene edges with the Benjamini-Hochberg false-discovery-rate procedure and call edges significant at an adjusted p-value < 0.05. The rationale is that the paired, non-parametric design matches the fact that perturbed and control cells are sampled in pairs from the same trained GRouNdGAN model (e.g., step_1000000.pth checkpoint), so per-cell differences test whether the imposed causal edge is expressed in the simulation. Finally, compute the fraction of significant edges per GRN and write per-edge results to knockout_edge_significance.csv.

## Related tools

- **GRouNdGAN** (Generates matched perturbed (TF knockout, value 0) and unperturbed cells from a trained checkpoint via the --perturb command, providing the paired expression samples tested with the Wilcoxon signed-rank test) — https://github.com/Emad-COMBINE-lab/GRouNdGAN

## Examples

```
from scipy.stats import wilcoxon; stat, p = wilcoxon(perturbed_expr, control_expr, alternative='two-sided')
```

## Evaluation signals

- Adjusted p-values must come from Benjamini-Hochberg FDR correction applied across all TF-gene edges, with significance called at adjusted p < 0.05
- The fraction of significant edges must reproduce ~66.5% for the GRNBoost2 GRN and ~75.2% for the PPCOR GRN on PBMC-CTL (within small numerical tolerance)
- Perturbed and control cells must be sampled from the same fixed batch to enable a valid matched case-control comparison
- Results must be written per-edge to knockout_edge_significance.csv

## Limitations

- The test only quantifies whether the imposed GRN is causally expressed in simulated cells; it does not predict real knockout outcomes
- Requires a trained GRouNdGAN model per dataset — the generative model must be retrained for new data
- The imposed GRN may not be the unique/most probable causal graph for the observational dataset, affecting interpretation of edge significance

## Evidence

- [methods] Perturbation via --perturb: "--perturb        Perform a perturbation experiment using a trained GRouNdGAN model"
- [abstract] In silico knockout purpose: "GRouNdGAN can synthesize cells under new conditions to perform in silico TF knockout experiments."
- [intro] Causal structure enables interventional sampling: "the causal structure of GRouNdGAN enables it to be used for sampling from both interventional and observational data distributions, enabling in silico knockout experiments."
