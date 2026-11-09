---
name: pseudotime-spearman-correlation-analysis
description: Use when a trajectory/pseudotime method has been run on scRNA-seq data
  with a known cell type ordering and you must quantify agreement per lineage and
  check consistency across multiple independent runs.
when_to_use_negative:
- When no independent information on the biological identity/ordering of cells exists
  (e.g., no validated marker-based ordering), so the correlation cannot be interpreted
- When the underlying RNA velocity estimation itself is faulty — trajectory inference
  and hence pseudotime validation are not meaningful
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Cytopath
  role: Produces the inferred pseudotime (via Markov chain simulation, dynamic time
    warping alignment, and alignment-score-based cell assignment) that is validated
    by Spearman correlation
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Preprocesses the data and estimates RNA velocity that Cytopath depends on
    as input for pseudotime inference
- name: velocyto
  role: Generates the spliced/unspliced/ambiguous count matrices (e.g., from Burns
    et al. GSE71982 reads) that feed velocity estimation
  repo: https://github.com/velocyto-team/velocyto-notebooks
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:e13b1a50f4e785f3254509026122a3bfd1f84451da3fdc011a223d791591a7fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cytopath
  - scvelo
  - velocyto
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# pseudotime-spearman-correlation-analysis

## Summary

Validates computationally inferred pseudotime (e.g., from Cytopath's simulation-based trajectory inference on RNA velocity fields) by computing the Spearman correlation between inferred pseudotime and a known ordering of cell types, repeated across independent runs to assess robustness.

## When to use

Use when a trajectory/pseudotime method has been run on scRNA-seq data with a known cell type ordering and you must quantify agreement per lineage and check consistency across multiple independent runs.

## When NOT to use

- When no independent information on the biological identity/ordering of cells exists (e.g., no validated marker-based ordering), so the correlation cannot be interpreted
- When the underlying RNA velocity estimation itself is faulty — trajectory inference and hence pseudotime validation are not meaningful

## Inputs

- Per-cell pseudotime vector from Cytopath trajectory-step assignment (one per lineage per run)
- Known/ranked ordering of cell types for the differentiation process (e.g., neonatal mouse inner ear cell type stages from Burns et al.)
- Multiple independent Cytopath runs (e.g., 10 initialisations) for robustness assessment

## Outputs

- Per-run Spearman correlation coefficient between inferred pseudotime and known cell type ordering, per lineage
- Mean Spearman correlation across independent runs
- Robustness verdict (high and consistent correlation across runs)

## How to apply

After Cytopath constructs a consensus trajectory and assigns each cell to a trajectory step via the alignment score of static and velocity profiles, extract the per-cell inferred pseudotime and the rank of the known cell type ordering (e.g., the Burns et al. neonatal mouse inner ear lineage stages). Compute the Spearman rank correlation between the two vectors separately for each lineage, since a convergent topology yields one trajectory per lineage. Repeat this over the independent initialisations (e.g., 10 runs) of the Markov-chain simulation ensemble and record per-run values plus the mean. Accept the result when the correlation is high and consistent across runs, as reported for Cytopath on datasets with known orderings (hippocampus: high Spearman correlation per lineage, robust across multiple runs).

## Related tools

- **Cytopath** (Produces the inferred pseudotime (via Markov chain simulation, dynamic time warping alignment, and alignment-score-based cell assignment) that is validated by Spearman correlation) — https://github.com/aron0093/cytopath
- **scvelo** (Preprocesses the data and estimates RNA velocity that Cytopath depends on as input for pseudotime inference)
- **velocyto** (Generates the spliced/unspliced/ambiguous count matrices (e.g., from Burns et al. GSE71982 reads) that feed velocity estimation) — https://github.com/velocyto-team/velocyto-notebooks

## Examples

```
from scipy.stats import spearmanr; rho, p = spearmanr(cytopath_pseudotime, known_celltype_rank)
```

## Evaluation signals

- Spearman correlation between inferred pseudotime and known cell type ordering is high for each lineage
- Per-run correlation values are consistent (robust) across the 10 independent initialisations; record per-run values and the mean
- Correlation is computed per trajectory/lineage rather than on the pooled convergent topology
- Pseudotime inference was robust to root/terminal probability thresholds in [0.85, 1], so correlation should not collapse under small threshold changes within that range

## Limitations

- Pseudotime is robust to root/terminal probability thresholds only within [0.85, 1]; robustness beyond this range is reported only as 'data not shown'
- If the underlying RNA velocity estimation is faulty (e.g., time-invariant rate assumptions violated), inferred pseudotime — and thus its correlation with known ordering — is not meaningful; spurious root/terminal states were observed on the erythroid gastrulation dataset
- Correctness of RNA velocity estimation cannot be verified in every scenario, even when marker-based biological information is available

## Evidence

- [results] High per-lineage correlation and multi-run robustness on the hippocampus dataset: "The Spearman correlation between pseudotime inferred by Cytopath for each trajectory to known ordering of cell types is high (Figure 2D) and robust across multiple runs (Figure 2E)."
- [methods] How cells receive pseudotime values (alignment score of static and velocity profiles): "Assignment is based on an alignment score that evaluates for each cell the similarity of its static as well as the velocity profile with each trajectory step."
- [discussion] Documented robustness range for the pseudotime being validated: "We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown)."
- [readme] Cytopath tool definition from the repository README: "Cytopath is a method for trajectory inference with single-cell RNA sequencing data."
