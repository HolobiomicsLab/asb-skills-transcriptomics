---
name: go-kegg-gene-set-enrichment-analysis
description: Use when you have gene modules or regulons (e.g., CEFCON RGMs or pySCENIC
  regulons) and need to quantify their biological meaningfulness via GO_BP/KEGG enrichment
  precision for benchmarking.
when_to_use_negative:
- When the goal is ranking individual genes or regulators rather than evaluating coherence
  of pre-detected gene modules
- When the input is already an enrichment result table and no module-level precision
  benchmark against a baseline (e.g., SCENIC) is required
- When assessing GRN edge inference accuracy (use AUROC/AUPRC-style GRN benchmarks
  instead of enrichment precision)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: CEFCON
  role: Produces the regulon-like gene modules (RGMs) via cefcon_results.RGM_activity()
    that are subjected to GO_BP/KEGG enrichment
  repo: https://github.com/WPZgithub/CEFCON
- name: pyscenic
  role: Baseline regulon-detection method (>= 0.12.0) whose regulons undergo the identical
    enrichment precision evaluation
- name: AUCell
  role: Computes the per-cell AUCell activity matrix (AUCell_mtx.csv) of RGMs using
    the aertslab AUCell scoring scheme, complementing the enrichment precision metric
  repo: https://github.com/aertslab/AUCell
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:bc2e6012d4c51b89dc2933e910d5253c5e1d69ea0c7eecacc9c9e256ca68e216
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - AUCell
  - CEFCON
  - pyscenic
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# go-kegg-gene-set-enrichment-analysis

## Summary

Perform GO_BP and KEGG functional enrichment on detected gene modules and compute enrichment precision as the fraction of modules significant at BH-adjusted p <= 0.05, used here to benchmark CEFCON's regulon-like gene modules (RGMs) against SCENIC regulons.

## When to use

Use when you have gene modules or regulons (e.g., CEFCON RGMs or pySCENIC regulons) and need to quantify their biological meaningfulness via GO_BP/KEGG enrichment precision for benchmarking.

## When NOT to use

- When the goal is ranking individual genes or regulators rather than evaluating coherence of pre-detected gene modules
- When the input is already an enrichment result table and no module-level precision benchmark against a baseline (e.g., SCENIC) is required
- When assessing GRN edge inference accuracy (use AUROC/AUPRC-style GRN benchmarks instead of enrichment precision)

## Inputs

- RGMs.csv — gene modules detected per lineage via cefcon_results.RGM_activity() from precomputed cell-lineage-specific GRNs (edge_threshold_avgDegree=8) for hESC, mESC, and three mHSC lineages
- pySCENIC regulons on the same five datasets (baseline)
- Gene sets and annotations for GO biological process and KEGG pathways

## Outputs

- Per-module GO_BP and KEGG enrichment results with BH-adjusted p-values
- Per-dataset enrichment precision (fraction of modules significant at BH-adjusted p <= 0.05) for CEFCON vs SCENIC
- Comparison table supporting Fig. 5b-d showing CEFCON's higher enrichment precision

## How to apply

For each detected regulon-like gene module, run GO_BP and KEGG functional enrichment on its member genes. Compute enrichment precision per dataset as the fraction of modules that reach significance at Benjamini-Hochberg-adjusted p <= 0.05. Apply the identical procedure to the pySCENIC (>=0.12.0) baseline regulons on the same hESC, mESC, and mHSC lineage datasets so precision values are directly comparable. Combine with per-cell AUCell activity scoring of the modules and downstream NMI clustering evaluation to confirm module quality, as reported in Fig. 5b-d of the CEFCON paper.

## Related tools

- **CEFCON** (Produces the regulon-like gene modules (RGMs) via cefcon_results.RGM_activity() that are subjected to GO_BP/KEGG enrichment) — https://github.com/WPZgithub/CEFCON
- **pyscenic** (Baseline regulon-detection method (>= 0.12.0) whose regulons undergo the identical enrichment precision evaluation)
- **AUCell** (Computes the per-cell AUCell activity matrix (AUCell_mtx.csv) of RGMs using the aertslab AUCell scoring scheme, complementing the enrichment precision metric) — https://github.com/aertslab/AUCell

## Examples

```
cefcon_results.RGM_activity()  # writes RGMs.csv; then run GO_BP/KEGG enrichment per module and compute precision at BH-adj p <= 0.05
```

## Evaluation signals

- Enrichment precision for CEFCON should exceed that of SCENIC on all five tested datasets (hESC, mESC, three mHSC lineages), consistent with Fig. 5b-d
- The compiled comparison table should report higher NMI from hierarchical clustering of the AUCell RGM-activity matrix versus reference cell-type labels for CEFCON than SCENIC
- A module counts as significant only at BH-adjusted p <= 0.05 — verify the multiple-testing correction is applied before computing precision
- Each lineage's modules should be written to RGMs.csv and its activity matrix to AUCell_mtx.csv before enrichment and clustering

## Limitations

- Enrichment precision depends on the quality of the upstream GRN and driver regulator identification; CEFCON edges were preselected with edge_threshold_avgDegree=8 before RGM detection
- CEFCON can only handle one cell lineage at a time, so module detection and enrichment must be run per lineage rather than across bifurcating trajectories simultaneously
- The benchmark result is specific to the five tested datasets; CEFCON's superiority may not transfer to other tissues or species

## Evidence

- [methods] Enrichment precision definition: [evidence span withheld — anchor status: not_found]
- [methods] RGM detection call: "# Identify regulon-like gene modules
cefcon_results.RGM_activity()"
- [abstract] Benchmarking superiority finding: "Extensive benchmarking tests consistently demonstrated the superiority of CEFCON in GRN construction, driver regulator identification, and gene module identification over baseline methods"
- [readme] AUCell usage for module activity: "Moreover, CEFCON detects gene regulatory modules (i.e., RGMs) involving the identified driver regulators and measure their activities using [AUCell]"
- [readme]  pyscenic version requirement: "- [pyscenic(>=0.12.0)](https://pyscenic.readthedocs.io/en/latest/installation.html)"
