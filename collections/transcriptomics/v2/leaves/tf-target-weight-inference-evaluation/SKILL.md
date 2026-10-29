---
name: tf-target-weight-inference-evaluation
description: Use when a synthetic benchmark has known signed TF-target weights (e.g.,
  10 TFs per target gene) and you must quantify recovery of regulatory weights, velocities,
  and edges via Spearman correlation, F1, and AUROC.
when_to_use_negative:
- Input is real scRNA-seq data without splicing information but with no ground-truth
  regulatory weights — there is nothing to validate the inferred weights against;
  use TFvelo's standard run/analysis demo workflow instead
- The goal is runtime benchmarking on large-scale datasets — TFvelo is known to be
  slower than scVelo, and this synthetic validation does not address efficiency
- Genes whose dynamics TFvelo is known to fit poorly (cf. Fig. S14) are the focus
  — weight recovery metrics may not be meaningful for those genes
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3308
tools:
- name: TFvelo
  role: Fits the regulatory dynamics model on the synthetic counts via generalized
    EM with correlation-based weight initialization and lsq_linear weight optimization,
    producing inferred TF weights and RNA velocities
  repo: github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Provides the vanilla EM dynamics-model baseline run on the same simulated
    counts for side-by-side metric comparison
  repo: theislab/scvelo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:c7a222d642cb43090e01bf29067f8b5848568159f476259b0083281b73c5ae11
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ScVelo
  - TFvelo
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# tf-target-weight-inference-evaluation

## Summary

Validates that TFvelo reconstructs ground-truth gene regulatory dynamics from synthetic multi-TF benchmark data by scoring inferred TF weights, RNA velocities, and thresholded TF–target edges against signed ground truth, and benchmarks them against scVelo's EM baseline.

## When to use

Use when a synthetic benchmark has known signed TF-target weights (e.g., 10 TFs per target gene) and you must quantify recovery of regulatory weights, velocities, and edges via Spearman correlation, F1, and AUROC.

## When NOT to use

- Input is real scRNA-seq data without splicing information but with no ground-truth regulatory weights — there is nothing to validate the inferred weights against; use TFvelo's standard run/analysis demo workflow instead
- The goal is runtime benchmarking on large-scale datasets — TFvelo is known to be slower than scVelo, and this synthetic validation does not address efficiency
- Genes whose dynamics TFvelo is known to fit poorly (cf. Fig. S14) are the focus — weight recovery metrics may not be meaningful for those genes

## Inputs

- Synthetic dataset: 200 simulated gene dynamics across 1000 cells with signed ground-truth TF regulatory weights (10 TFs per target gene), per Supplementary Section 3
- Simulated count matrix compatible with TFvelo/scVelo AnnData input (.h5ad)
- Signed ground-truth TF–target edge list for edge-level classification

## Outputs

- Spearman correlation between inferred and ground-truth TF weights (target ≈ 0.823)
- Spearman correlation between inferred and ground-truth RNA velocities (target ≈ 0.894)
- Edge-level F1 for thresholded TF–target edges (target ≈ 0.944)
- AUROC for TF–target edge classification (target ≈ 0.962)
- Side-by-side metrics table of TFvelo vs. vanilla-EM baseline (Fig. S4 comparison panel)

## How to apply

Generate the Supplementary Section 3 synthetic benchmark: 200 simulated gene dynamics across 1000 cells, with each target gene regulated by 10 transcription factors carrying signed ground-truth regulatory weights. Run TFvelo with default settings (generalized EM with correlation-based weight initialization via --init_weight_method, lsq_linear weight optimization via --WX_method, and E-step time assignment over --n_time_points) on the simulated counts, and run scVelo's EM dynamics model as the vanilla baseline. After ~20 optimization iterations, compute Spearman correlation between inferred and ground-truth TF weights (expect ≈0.823) and between inferred and ground-truth RNA velocities (expect ≈0.894). Threshold the inferred weights to classify TF–target edges and compute F1 (≈0.944) and AUROC (≈0.962) against the signed ground-truth edges. Compile TFvelo versus vanilla-EM metrics side by side to reproduce the Fig. S4 comparison panel.

## Related tools

- **TFvelo** (Fits the regulatory dynamics model on the synthetic counts via generalized EM with correlation-based weight initialization and lsq_linear weight optimization, producing inferred TF weights and RNA velocities) — github.com/xiaoyeye/TFvelo
- **ScVelo** (Provides the vanilla EM dynamics-model baseline run on the same simulated counts for side-by-side metric comparison) — theislab/scvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name synthetic --init_weight_method correlation --WX_method lsq_linear --max_iter 20
```

## Evaluation signals

- Metrics table contains Spearman weight correlation, Spearman velocity correlation, F1, and AUROC columns for both TFvelo and the vanilla-EM baseline
- TFvelo Spearman weight correlation ≈ 0.823 within reproducibility tolerance
- TFvelo Spearman velocity correlation ≈ 0.894 within reproducibility tolerance
- TFvelo F1 ≈ 0.944 and AUROC ≈ 0.962 against signed ground-truth edges
- TFvelo exceeds the vanilla-EM baseline on every metric in the comparison table

## Limitations

- TFvelo has lower time efficiency than scVelo, a weakness for large-scale datasets; running with more CPUs in parallel (via --n_jobs) can reduce runtime
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14), and improving modeling for more genes remains challenging
- The current linear model for estimating TF weights may be insufficient; no implementation for alternative models is provided

## Evidence

- [results] Synthetic validation targets: [evidence span withheld — anchor status: not_found]
- [intro] Model rationale: "we propose TFvelo to model the RNA velocity based on the expression levels of TFs, where the velocity refers to the changing rate of RNA abundance"
- [readme] Default hyperparameters: "--init_weight_method: The method to initialize the weights. Correlation is adopted by default. --WX_method: The method to optimize weight. lsq_linear is adopted by default."
- [methods] EM time assignment: "--n_time_points: The number of time points in the time assinment (E step of the generalized EM algorithm)."
- [discussion] Synthetic dataset purpose: "Experiments on a synthetic dataset validate that TFvelo could detect the underlying dynamics from the data"
