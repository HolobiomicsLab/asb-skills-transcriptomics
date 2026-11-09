---
name: permutation-null-significance-testing
description: Use when detected network modules (e.g. fRNC global/local RNCs from ESCA
  TCGA data) must be validated against a random null to confirm their scores significantly
  exceed chance at the detection FDR threshold (1e-5).
when_to_use_negative:
- Modules were not detected with fRNC's runmodule scoring scheme, so random node sets
  cannot be scored comparably
- The analysis goal is differential expression testing rather than module robustness
  against random node sets
- Computational budget cannot accommodate 100,000 random node-set score evaluations
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3517
tools:
- name: fRNC
  role: R package supplying runmodule, combinp, and interStringency used to score
    random node sets with the same scheme applied during module search
  repo: https://github.com/leiming8886/fRNC
- name: R
  role: Environment in which the 100,000-permutation null sampling and empirical p-value
    calculation are executed
  repo: https://github.com/BioinformaticsSTU/fRNC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:6009768718893fb30761b3af6f072ac8984917c06cf29dbcd8e500e219363321
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
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# permutation-null-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assesses whether detected RBP-ncRNA circuit (RNC) modules are significantly more robust than random by building an empirical null distribution from 100,000 randomly sampled node sets matched to each module's size, then computing a one-sided empirical p-value for the module's observed summary score.

## When to use

Use when detected network modules (e.g. fRNC global/local RNCs from ESCA TCGA data) must be validated against a random null to confirm their scores significantly exceed chance at the detection FDR threshold (1e-5).

## When NOT to use

- Modules were not detected with fRNC's runmodule scoring scheme, so random node sets cannot be scored comparably
- The analysis goal is differential expression testing rather than module robustness against random node sets
- Computational budget cannot accommodate 100,000 random node-set score evaluations

## Inputs

- Previously detected ESCA global and local RNC modules from fRNC runmodule output
- gene2weight node scores from combinp (edgeR DEG logFC and PValue for miRNA, mRNA, lncRNA)
- RBP-ncRNA interaction network (interac_rbp from interStringency, high stringency CLIP evidence >= 3)

## Outputs

- permutation_significance.csv with module size, observed summary score, null distribution mean and standard deviation, and one-sided empirical p-value per module

## How to apply

In R with the fRNC package, load the previously detected ESCA global and local RNC modules together with their gene-to-weight scores (gene2weight built via combinp from edgeR differential expression results). Randomly sample 100,000 node sets from the RBP-ncRNA network (interac_rbp from interStringency with high stringency, CLIP evidence >= 3), each set matching the size of the corresponding detected module. Score every random node set with the same scoring scheme fRNC uses during module search (runmodule with method = "global", FDR = 1e-5), then build the empirical null distribution of summary scores. Compute a one-sided empirical p-value for each real module's score against this null and write module size, observed score, null mean, null standard deviation, and p-value to permutation_significance.csv.

## Related tools

- **fRNC** (R package supplying runmodule, combinp, and interStringency used to score random node sets with the same scheme applied during module search) — https://github.com/leiming8886/fRNC
- **R** (Environment in which the 100,000-permutation null sampling and empirical p-value calculation are executed) — https://github.com/BioinformaticsSTU/fRNC

## Examples

```
res.list_global <- runmodule(network = interac_rbp, gene2weight, method = "global", FDR = 1e-5)
```

## Evaluation signals

- Each module's empirical p-value falls below the 1e-5 FDR threshold used in runmodule module detection
- Observed module summary score lies in the extreme right tail of the 100,000-sample null distribution
- Null distribution mean and standard deviation are finite and consistent with random node sets of the same size
- permutation_significance.csv contains one row per detected module with size, observed score, null mean, null SD, and p-value columns populated

## Limitations

- Empirical p-values are bounded by the number of permutations (1/resolution limit of 100,000 samples)
- The null model samples node sets ignoring network connectivity, so it tests score enrichment rather than topological robustness
- Computational cost scales with the 100,000 permutations and module scoring, which may be slow for large networks

## Evidence
