---
name: synthetic-gene-dynamics-simulation
description: Use when validating an RNA velocity or gene regulation model on data
  with known ground truth, specifically to test recovery of signed TF-target regulatory
  weights and RNA velocity from simulated counts.
when_to_use_negative:
- The dataset is real scRNA-seq without splicing information and the goal is biological
  trajectory inference rather than ground-truth method validation — use the TFvelo
  application pipeline instead
- Input data already contains measured spliced/unspliced counts for a real tissue
  (e.g., pancreas), where the empirical scVelo/TFvelo comparison is the appropriate
  skill
- The research question concerns runtime scalability on large datasets; TFvelo is
  slower than scVelo, a known weakness for large-scale data
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: TFvelo
  role: 'Method under test: fits gene dynamics from TF expression via generalized
    EM with correlation-based weight initialization and lsq_linear weight optimization,
    recovering TF weights and RNA velocity from simulated counts'
  repo: github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Vanilla EM baseline (expectation-maximization dynamics model) run on the same
    simulated counts for side-by-side metric comparison
  repo: theislab/scvelo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:fbe0adae356d1074f93fe32432e55e61a129e4e08b98035904dc7c850a8fb01f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ScVelo
  - TFvelo
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# synthetic-gene-dynamics-simulation

## Summary

Generates and uses a synthetic benchmark of 200 simulated gene dynamics across 1000 cells, where each target gene is regulated by 10 transcription factors with signed ground-truth weights, to validate that TFvelo can recover the underlying dynamic model and TF-target regulatory relationships.

## When to use

Use when validating an RNA velocity or gene regulation model on data with known ground truth, specifically to test recovery of signed TF-target regulatory weights and RNA velocity from simulated counts.

## When NOT to use

- The dataset is real scRNA-seq without splicing information and the goal is biological trajectory inference rather than ground-truth method validation — use the TFvelo application pipeline instead
- Input data already contains measured spliced/unspliced counts for a real tissue (e.g., pancreas), where the empirical scVelo/TFvelo comparison is the appropriate skill
- The research question concerns runtime scalability on large datasets; TFvelo is slower than scVelo, a known weakness for large-scale data

## Inputs

- Synthetic benchmark dataset: 200 simulated gene dynamics across 1000 cells with 10 TFs per target gene and signed ground-truth regulatory weights (Supplementary Section 3)
- Signed ground-truth TF-target edge list and ground-truth RNA velocity vectors
- Simulated count matrices consumable by TFvelo and scVelo

## Outputs

- Inferred TF regulatory weight matrix per target gene
- Inferred RNA velocity estimates per cell
- Metrics table: Spearman weight correlation, Spearman velocity correlation, F1, and AUROC for TFvelo vs. vanilla-EM baseline
- Fig. S4-style comparison panel of TFvelo vs. baseline

## How to apply

Generate the synthetic benchmark specified in Supplementary Section 3: 200 simulated gene dynamics over 1000 cells, with each target gene regulated by 10 transcription factors carrying signed ground-truth regulatory weights. Run TFvelo on the simulated counts with default settings, i.e., generalized EM with correlation-based weight initialization (--init_weight_method), lsq_linear weight optimization (--WX_method), and E-step time assignment over n_time_points. Run the vanilla EM baseline (scVelo's expectation-maximization dynamics model) on the same simulated counts for comparison. After 20 optimization iterations, quantify recovery by Spearman correlation between inferred and ground-truth TF weights (target ~0.823) and between inferred and ground-truth RNA velocities (target ~0.894). Threshold the inferred regulatory weights to classify TF-target edges and compute F1 (~0.944) and AUROC (~0.962) against the signed ground-truth edges, compiling TFvelo vs. vanilla-EM metrics side by side as in the Fig. S4 comparison panel.

## Related tools

- **TFvelo** (Method under test: fits gene dynamics from TF expression via generalized EM with correlation-based weight initialization and lsq_linear weight optimization, recovering TF weights and RNA velocity from simulated counts) — github.com/xiaoyeye/TFvelo
- **ScVelo** (Vanilla EM baseline (expectation-maximization dynamics model) run on the same simulated counts for side-by-side metric comparison) — theislab/scvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name synthetic --init_weight_method correlation --WX_method lsq_linear --max_iter 20
```

## Evaluation signals

- Spearman correlation between ground-truth and inferred TF weights ≈ 0.823 after 20 optimization iterations
- Spearman correlation between ground-truth and inferred RNA velocities ≈ 0.894
- Edge-level F1 against signed ground-truth TF-target edges ≈ 0.944
- Edge-level AUROC against signed ground-truth edges ≈ 0.962
- TFvelo exceeds the vanilla-EM (scVelo) baseline on every metric (weight correlation, velocity correlation, F1, AUROC) in the comparison table

## Limitations

- TFvelo's time efficiency is lower than baseline approaches in the same framework (e.g., scVelo); running with more CPUs in parallel can reduce runtime
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14); improving modeling coverage of more genes remains a challenge
- The current linear model for estimating TF weights may be insufficient; no alternative implementations are provided

## Evidence

- [discussion] Experiments on a synthetic dataset validate that TFvelo could detect the underlying dynamics from the data: "Experiments on a synthetic dataset validate that TFvelo could detect the underlying dynamics from the data"
- [methods] generalized EM with correlation-based weight initialization: "--init_weight_method: The method to initialize the weights. Correlation is adopted by default."
- [readme] lsq_linear weight optimization: "--WX_method: The method to optimize weight. lsq_linear is adopted by default."
- [readme] E-step time assignment over n_time_points: "--n_time_points: The number of time points in the time assinment (E step of the generalized EM algorithm)."
- [intro] ScVelo EM dynamics baseline: "ScVelo10 models the dynamics without the steady state assumption and employs an Expectation-Maximization (EM) approach"
