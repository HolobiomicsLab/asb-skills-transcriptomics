---
name: rbp-ncrna-network-module-scoring
description: Use when detected RNC modules (global or local, e.g. from runmodule with
  FDR = 1e-5) need permutation-based validation of whether their summary scores exceed
  what random node sets of the same size would achieve.
when_to_use_negative:
- The modules have not yet been detected by fRNC runmodule — build the null against
  existing observed scores, not before module detection
- The scoring scheme (gene2weight or network) differs from that used during module
  search — the null must use the identical scoring to be a valid comparison
- The goal is functional/GO enrichment of modules rather than statistical robustness
  of module scores — permutation of node sets does not assess enrichment
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: fRNC
  role: R package supplying the module scoring scheme (runmodule), the gene2weight
    node scoring (combinp), and the RBP-ncRNA network (interStringency) used to generate
    both observed module scores and the permutation null
  repo: https://github.com/leiming8886/fRNC
- name: R
  role: Environment in which the 100,000-permutation null sampling, empirical p-value
    computation, and CSV export are scripted
  repo: https://github.com/BioinformaticsSTU/fRNC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:22ec579945a4df37e7e540b716d19ae8e60baff40e5ad10d320b17515842fcde
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/fRNC
  repo_url: https://github.com/leiming8886/fRNC
  tools:
  - R
  - fRNC
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# rbp-ncrna-network-module-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess the statistical significance of detected RBP-ncRNA circuit (RNC) modules by building an empirical null distribution of module scores from 100,000 randomly sampled node sets of matching size, and computing one-sided empirical p-values against the observed scores. This confirms that fRNC-detected modules score significantly higher than random node sets under the same scoring scheme.

## When to use

Use when detected RNC modules (global or local, e.g. from runmodule with FDR = 1e-5) need permutation-based validation of whether their summary scores exceed what random node sets of the same size would achieve.

## When NOT to use

- The modules have not yet been detected by fRNC runmodule — build the null against existing observed scores, not before module detection
- The scoring scheme (gene2weight or network) differs from that used during module search — the null must use the identical scoring to be a valid comparison
- The goal is functional/GO enrichment of modules rather than statistical robustness of module scores — permutation of node sets does not assess enrichment

## Inputs

- Detected ESCA global and local RNC modules from fRNC runmodule (method = "global" and method = "local", FDR = 1e-5)
- gene-to-weight score table (gene2weight) from combinp applied to edgeR DEG results (type, logFC, PValue columns)
- RBP-ncRNA interaction network (interac_rbp) from interStringency, high stringency ENCORI interactions with CLIP evidence >= 3

## Outputs

- permutation_significance.csv containing module size, observed summary score, null distribution mean and standard deviation, and one-sided empirical p-value per module
- Empirical null distributions of summary scores from 100,000 random node sets per module

## How to apply

In R, load the previously detected ESCA global and local RNC modules together with their gene-to-weight scores (gene2weight produced by combinp from edgeR differential-expression results). Randomly sample 100,000 node sets from the RBP-ncRNA network (interac_rbp from interStringency with high stringency, CLIP evidence >= 3), each set matching the size of the corresponding detected module. Compute the summary node score of every random set using the same scoring scheme fRNC applies during module search (runmodule with method = "global", FDR = 1e-5). Build the empirical null distribution of summary scores and calculate a one-sided empirical p-value for each real module's observed score. Write module size, observed summary score, null mean and standard deviation, and empirical p-value to permutation_significance.csv.

## Related tools

- **fRNC** (R package supplying the module scoring scheme (runmodule), the gene2weight node scoring (combinp), and the RBP-ncRNA network (interStringency) used to generate both observed module scores and the permutation null) — https://github.com/leiming8886/fRNC
- **R** (Environment in which the 100,000-permutation null sampling, empirical p-value computation, and CSV export are scripted) — https://github.com/BioinformaticsSTU/fRNC

## Examples

```
R: res.list_global <- runmodule(network = interac_rbp, gene2weight, method = "global", FDR = 1e-5); # then sample 100000 random node sets of equal size, score them with the same scheme, and write permutation_significance.csv
```

## Evaluation signals

- Each module's empirical p-value falls below the 1e-5 FDR threshold used during module detection (runmodule, method = "global", FDR = 1e-5)
- Observed module summary score exceeds the null distribution mean by a margin consistent with a one-sided p < 1e-5 against 100,000 random node sets
- Each random node set matches the size of its corresponding detected module — permutation_significance.csv module sizes equal the detected module sizes
- permutation_significance.csv contains all required columns: module size, observed summary score, null mean, null standard deviation, empirical p-value

## Limitations

- The null is score-based only: random node sets ignore the connectivity structure of the RBP-ncRNA network, so the test addresses score robustness rather than topology
- Results depend on reusing the exact same scoring scheme and parameters (e.g. FDR = 1e-5, high-stringency interactions) as the original module search; changing them invalidates the comparison
- Discussion-section content and some metadata were unavailable in the source material, so no additional caveats from the authors' discussion could be extracted

## Evidence

- [results] The robustness of identified modules was tested by randomly sampling 100,000 node sets and checking whether the module's score was significantly higher than that of a random RNC.: [evidence span withheld — anchor status: not_found]
