---
name: adjusted-rand-index-clustering-evaluation
description: Use when clustering or community-detection predictions must be scored
  against known true labels (e.g. z_mus_dynamic_true from simulate_mus_dynamic_dcbm)
  across multiple subjects and time points.
when_to_use_negative:
- No ground-truth labels exist (e.g. real scRNA-seq networks) — ARI cannot be computed;
  use stability or biological-enrichment assessments instead
- Comparing clusterings with different numbers of predicted and true labels is fine
  for ARI, but raw (unadjusted) Rand or accuracy metrics would be misleading and should
  not be substituted for ARI
- Input is a single static clustering with no multi-subject/multi-time-point structure
  — the aggregation scheme (mean over subjects and time points) does not apply as
  designed here
edam_operation: http://edamontology.org/operation_0563
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3473
tools:
- name: MuDCoD
  role: Produces the predicted communities (pred_MuDCoD) whose ARI against true labels
    is scored; documentation shows explicit ARI comparison and mean computation
  repo: https://github.com/bo1929/MuDCoD
- name: MuSDynamicDCBM
  role: Simulation model that supplies z_mus_dynamic_true ground-truth labels needed
    for ARI computation
  repo: https://github.com/bo1929/MuDCoD
- name: numpy
  role: Used for array manipulation and computing/aggregating mean ARI values over
    subjects and time points
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:062fcd21debd88a6f3241000e7c4242da1b7e0a32592bbfa0427832fb6ea3a9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - MuDCoD
  - MuSDynamicDCBM
  - numpy
  edam_operation: http://edamontology.org/operation_0563
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# adjusted-rand-index-clustering-evaluation

## Summary

Evaluate community-detection accuracy by computing the adjusted Rand index (ARI) between predicted community assignments and ground-truth labels, per subject and time point, and aggregating mean ARI over replicates. In the MuDCoD workflow this is the standard metric for comparing methods on simulated MuSDynamicDCBM networks.

## When to use

Use when clustering or community-detection predictions must be scored against known true labels (e.g. z_mus_dynamic_true from simulate_mus_dynamic_dcbm) across multiple subjects and time points.

## When NOT to use

- No ground-truth labels exist (e.g. real scRNA-seq networks) — ARI cannot be computed; use stability or biological-enrichment assessments instead
- Comparing clusterings with different numbers of predicted and true labels is fine for ARI, but raw (unadjusted) Rand or accuracy metrics would be misleading and should not be substituted for ARI
- Input is a single static clustering with no multi-subject/multi-time-point structure — the aggregation scheme (mean over subjects and time points) does not apply as designed here

## Inputs

- pred_MuDCoD predicted community array (S x T x n)
- z_mus_dynamic_true ground-truth community array (S x T x n) from MuSDynamicDCBM simulation
- predictions from baseline methods (PisCES, Betzel-2019, multi-subject baseline) on the same networks

## Outputs

- Per-subject and per-time-point ARI values
- Mean ARI per method and parameter configuration (aggregated over replicates)
- ARI table file (e.g. ari_results.csv or ari_comparison.csv)

## How to apply

After running MuDCoD.fit_predict (e.g. with alpha=0.05*np.ones((T,2)), beta=0.05*np.ones(S), max_K=10, n_iter=30) on a simulated S x T x n x n adjacency tensor, compare the predicted communities to the true labels z_mus_dynamic_true (S x T x n arrays) using the adjusted Rand index. Compute ARI per subject and per time point, then average over subjects, time points, and replicates (e.g. 100 replicates per parameter configuration) to obtain mean ARI per method and configuration. Repeat identically for baseline methods (PisCES, Betzel-2019, static spectral clustering) so comparisons are on the same simulated networks, and write the aggregated table (e.g. ari_results.csv or ari_comparison.csv) for ARI-vs-parameter curves sweeping r_time, r_subject, and T.

## Related tools

- **MuDCoD** (Produces the predicted communities (pred_MuDCoD) whose ARI against true labels is scored; documentation shows explicit ARI comparison and mean computation) — https://github.com/bo1929/MuDCoD
- **MuSDynamicDCBM** (Simulation model that supplies z_mus_dynamic_true ground-truth labels needed for ARI computation) — https://github.com/bo1929/MuDCoD
- **numpy** (Used for array manipulation and computing/aggregating mean ARI values over subjects and time points)

## Examples

```
from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(pred_MuDCoD.ravel(), z_mus_dynamic_true.ravel()); print(f"mean(ARI) of MuDCoD: {ari_MuDCoD/T/S}")
```

## Evaluation signals

- ARI of a perfectly recovered partition equals 1.0; values near 0 indicate chance-level agreement with z_mus_dynamic_true
- Mean ARI is symmetric to averaging order — recomputing the mean over subjects/time points/100 replicates in different orders must give identical results
- Reproduced ARI-vs-parameter curves should match Fig. 3 patterns: MuDCoD gains >20% mean ARI over PisCES at SSoT with r_time=0 and T=8, and Betzel-2019 yields mean ARI <0.2 at r_subject=0.5 and/or r_time=0.5
- Per-subject/per-time-point ARI arrays must have shape S x T (e.g. 16 x 8) matching the label arrays before aggregation
- Running the identical fit_predict and ARI computation twice with the same simulated tensor should give identical ARI values (deterministic given fixed labels)

## Limitations

- The article reports no concrete numeric ARI values in its text — the benchmark code computes and prints ARI but published outputs must be re-derived from the validation patterns (Fig. 3 curves)
- ARI is only defined against simulated ground truth (z_mus_dynamic_true); on real scRNA-seq-derived networks there is no true labeling to compare against
- The paper contains no discussion section text and no changelog, limiting documented caveats about metric interpretation

## Evidence

- [methods] Compare predicted communities with true labels to evaluate accuracy: "You can compare `pred_MuDCoD` and `z_mus_dynamic_true` to evaluate the accuracy"
- [results] Mean ARI computation over subjects and time points: "print(f"mean(ARI) of MuDCoD: {ari_MuDCoD/T/S}")"
- [other] Benchmark compares MuDCoD against baseline methods by ARI on simulated networks: [evidence span withheld — anchor status: not_found]
- [abstract] MuDCoD robustness claim underlying the metric comparison: "MuDCoD effectively leverages apparent shared signals among networks of the subjects at individual time points, and performs robustly when there is no or little information sharing among the networks"
