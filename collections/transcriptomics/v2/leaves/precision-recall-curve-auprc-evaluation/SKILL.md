---
name: precision-recall-curve-auprc-evaluation
description: Use when a reconstructed GRN (TF/Gene/importance rankings) must be scored
  against an imposed or curated ground truth GRN, and ranked edges need precision-recall
  quantification with a random-predictor baseline.
when_to_use_negative:
- The ground truth GRN is unknown or merely a curated database without context-specific
  validation — the article notes curated databases are not ideal benchmarks due to
  context specificity of gene regulation
- The goal is evaluating simulated data quality (gene identity, trajectory, pseudo-time
  preservation) rather than GRN inference performance — use GRouNdGAN's --evaluate
  instead
- The simulated data were generated without any imposed GRN, so there is no gold standard
  against which to compute PR/AUPRC
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: GRouNdGAN
  role: Provides --benchmark_grn to evaluate a reconstructed GRN against the ground
    truth GRN with compute pr = True and save PR_curve.png
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers TF-gene rankings (k = 15 top TFs per gene) whose output is formatted
    for PR/AUPRC benchmarking
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GENIE3
  role: One of the GRN inference methods whose reconstructed networks are scored by
    AUPRC against the imposed GRN
- name: PIDC
  role: One of the GRN inference methods whose reconstructed networks are scored by
    AUPRC against the imposed GRN
- name: PPCOR
  role: One of the GRN inference methods whose reconstructed networks are scored by
    AUPRC against the imposed GRN
- name: BEELINE
  role: Reference benchmarking framework whose experimental results GRouNdGAN's benchmarking
    results are aligned with
  repo: https://github.com/Murali-group/Beeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:e0dff948aeed626cba96f7a3cdd80608ba741ea116d75696aa522f187afbb684
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - BEELINE
  - GENIE3
  - GRNBoost2
  - GRouNdGAN
  - PIDC
  - PPCOR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# precision-recall-curve-auprc-evaluation

## Summary

Evaluate reconstructed gene regulatory networks against a known ground truth by computing a global Precision-Recall curve and its AUPRC, as done in GRouNdGAN's --benchmark_grn step to score GRN inference methods on imposed control GRNs.

## When to use

Use when a reconstructed GRN (TF/Gene/importance rankings) must be scored against an imposed or curated ground truth GRN, and ranked edges need precision-recall quantification with a random-predictor baseline.

## When NOT to use

- The ground truth GRN is unknown or merely a curated database without context-specific validation — the article notes curated databases are not ideal benchmarks due to context specificity of gene regulation
- The goal is evaluating simulated data quality (gene identity, trajectory, pseudo-time preservation) rather than GRN inference performance — use GRouNdGAN's --evaluate instead
- The simulated data were generated without any imposed GRN, so there is no gold standard against which to compute PR/AUPRC

## Inputs

- Reconstructed GRN formatted as a three-column TF/Gene/importance CSV from GRNBoost2, GENIE3, PIDC, or PPCOR
- Ground truth GRN used for simulation (causal_graph.pkl nested dictionary of gene-index to TF-index sets)
- GRouNdGAN config file with [GRN Benchmarking] section (compute pr = True, plots save path, optional ground truth save path)

## Outputs

- Global Precision-Recall curve plot (PR_curve.png) saved to the plots save path
- AUPRC value per evaluated GRN inference method
- Optional exported ground truth GRN as CSV

## How to apply

Format each reconstructed GRN as a three-column TF/Gene/importance CSV (e.g., from GRNBoost2, GENIE3, PIDC, or PPCOR run on GRouNdGAN-simulated data), then run GRouNdGAN's --benchmark_grn with compute pr = True to compute a global Precision-Recall curve of inferred edges against the ground truth causal_graph (the imposed GRN used for simulation). Compare the AUPRC to the random-predictor baseline: in the Fig. 3 positive/negative even/odd-parity control GRN design, imposed edges should reach AUPRC far above random while unimposed edges score near random. Optionally export the ground truth GRN as CSV via the [GRN Benchmarking]/ground truth save path field, and save the PR_curve.png plot to the plots save path. Success is judged by high AUPRC for the imposed (positive control) GRN relative to the random baseline, reproducing the Fig. 3 / Supplementary Data 3 pattern.

## Related tools

- **GRouNdGAN** (Provides --benchmark_grn to evaluate a reconstructed GRN against the ground truth GRN with compute pr = True and save PR_curve.png) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers TF-gene rankings (k = 15 top TFs per gene) whose output is formatted for PR/AUPRC benchmarking) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GENIE3** (One of the GRN inference methods whose reconstructed networks are scored by AUPRC against the imposed GRN)
- **PIDC** (One of the GRN inference methods whose reconstructed networks are scored by AUPRC against the imposed GRN)
- **PPCOR** (One of the GRN inference methods whose reconstructed networks are scored by AUPRC against the imposed GRN)
- **BEELINE** (Reference benchmarking framework whose experimental results GRouNdGAN's benchmarking results are aligned with) — https://github.com/Murali-group/Beeline

## Examples

```
groundgan --benchmark_grn  # with [GRN Benchmarking] compute pr = True in the config, evaluating reconstructed GRN CSVs against causal_graph.pkl ground truth
```

## Evaluation signals

- Reconstructed GRNs from GRouNdGAN-simulated data show much higher AUPRC for imposed (positive control) edges than the same methods achieve on the original reference data
- Edges belonging to the unimposed (negative control) GRN receive near-random AUPRC scores, matching the random-predictor baseline
- PR_curve.png is produced in the configured plots save path when compute pr = True
- Benchmark results are qualitatively aligned with BEELINE's experimental results (Fig. 3 / Supplementary Data 3 pattern)

## Limitations

- The underlying causal graph of an observational dataset is not necessarily unique; verifying that the imposed GRN is the most probable one is beyond the study's scope
- Benchmarking results on PBMC-All are not conclusive, and follow-up studies are required to fully characterize the effect of distinct cell types on GRN inference performance
- GRouNdGAN only supports imposing bipartite GRNs (TF-gene), not multi-layer GRNs or TF-gene-phenotype causal graphs

## Evidence

- [methods] benchmark_grn command evaluates the reconstructed GRN against ground truth: "--benchmark_grn  Evaluate the performance of a GRN inference method in inferring the ground truth GRN"
- [abstract] GRouNdGAN bridges gap providing gold standard ground truth GRNs: "GRouNdGAN effectively bridges the existing gap between simulated and biological data benchmarks of GRN inference algorithms, providing gold standard ground truth GRNs"
- [discussion] recommend imposing subset of edges inferred from reference dataset: "we recommend first finding a set of regulatory edges from the reference dataset (e.g., using GRNBoost2 or even other data modalities) and imposing a selected subset of those in the simulated data"
- [intro] benchmarking objective aligned with BEELINE results: [evidence span withheld — anchor status: not_found]
- [readme] README benchmarking evaluates reconstructed GRN against ground truth used for simulation: "This command evaluates the reconstructed GRN (from GRouNdGAN-simulate data) against the ground truth GRN (used for simulation)."
