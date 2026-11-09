---
name: synthetic-differentiation-trajectory-simulation
description: Use when you need a controlled single-cell count dataset with known data-generating
  expression rates and batch-confounded cell states to test whether a batch-correction
  or disentanglement method recovers the truth.
when_to_use_negative:
- The goal is to evaluate batch correction on real, uncontrolled data — the simulation's
  ground-truth rates exist only for synthetic trajectories and cannot be verified
  against real atlases.
- Cell populations are not well-defined or cannot be split into control and batch-confounded
  groups, since the simulation's rationale requires a control group present in both
  treatment and control batches.
- The model under test cannot ingest one-hot covariate/batch indicators, because the
  batch-confounding design of the simulated system cannot be expressed.
tools:
- name: Frankencell
  role: Generates the synthetic differentiation trajectories by mixing reads from
    individual cells sampled from distinct, well-defined cell populations
- name: CODAL
  role: Downstream model trained on the simulated counts (with MINE mutual information
    regularization) to test recovery of ground-truth expression rates
  repo: github.com/cistrome/MIRA
- name: MIRA
  role: Topic-model framework on which CODAL's model architecture extends; provides
    the topic model trained on the simulated counts
  repo: github.com/cistrome/MIRA
- name: PyTorch
  role: Implementation backend for the VAE/topic-model training on simulated counts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:9e9807081542317de0837de455a863343dce862227b07c569e271cdf87373d48
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CODAL
  - Frankencell
  - MIRA
  - PyTorch
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# synthetic-differentiation-trajectory-simulation

## Summary

Generate synthetic differentiation trajectories with known ground-truth expression rates (e.g., a bifurcating system sampled into two batches with a batch-confounded gene A), as the paper does with Frankencell, to benchmark whether batch-corrected topic models like CODAL recover true biological expression.

## When to use

Use when you need a controlled single-cell count dataset with known data-generating expression rates and batch-confounded cell states to test whether a batch-correction or disentanglement method recovers the truth.

## When NOT to use

- The goal is to evaluate batch correction on real, uncontrolled data — the simulation's ground-truth rates exist only for synthetic trajectories and cannot be verified against real atlases.
- Cell populations are not well-defined or cannot be split into control and batch-confounded groups, since the simulation's rationale requires a control group present in both treatment and control batches.
- The model under test cannot ingest one-hot covariate/batch indicators, because the batch-confounding design of the simulated system cannot be expressed.

## Inputs

- Count matrices from scRNA-seq (or scRNA-seq+scATAC-seq) in AnnData/h5ad form, mixed from individual cells sampled from distinct, well-defined reference cell populations
- One-hot batch indicator covariates marking each simulated cell's batch of origin
- Simulated ground-truth expression rates for the injected gene A (defined per cell type per batch, upregulated only after the bifurcation point in the treatment batch)

## Outputs

- Synthetic bifurcating differentiation trajectory of single-cell counts distributed across two batches
- Ground-truth expression-rate table for gene A across batches and cell types, used to score fitted models
- Per-model predicted expression rates and mutual information estimates between expression-rate and technical-effect predictions (e.g., rendered as a Fig. 1e-style comparison figure)

## How to apply

Simulate a differentiation trajectory — e.g., a bifurcating system — sampled into two batches, injecting a data-generating gene A that is upregulated after the bifurcation point only in the treatment batch, and attach one-hot batch indicator covariates as proxies for hidden batch-specific factors. Frankencell-style simulation mixes reads from individual cells sampled from distinct, well-defined cell populations to build the synthetic trajectory. Split cells into control (unaffected by perturbation, present in both batches) and perturbation-induced (batch-confounded) populations so that ground-truth rates are defined per cell type per batch. Train candidate models (vanilla ELBO topic model vs. CODAL with MINE mutual information regularization) on the simulated counts, then compare each model's predicted expression rates for gene A against the simulated ground-truth rates across both batches. Retain the one-hot batch covariates throughout so technical-effect predictions can be evaluated independently of expression-rate predictions.

## Related tools

- **Frankencell** (Generates the synthetic differentiation trajectories by mixing reads from individual cells sampled from distinct, well-defined cell populations)
- **CODAL** (Downstream model trained on the simulated counts (with MINE mutual information regularization) to test recovery of ground-truth expression rates) — github.com/cistrome/MIRA
- **MIRA** (Topic-model framework on which CODAL's model architecture extends; provides the topic model trained on the simulated counts) — github.com/cistrome/MIRA
- **PyTorch** (Implementation backend for the VAE/topic-model training on simulated counts)

## Evaluation signals

- CODAL-fit predicted expression rates for gene A match the data-generating ground-truth rates across both batches, whereas the ELBO-only model's predictions are entangled with batch.
- Mutual information estimates between expression-rate and technical-effect predictions are low for the disentangled model, confirming batch information is not encoded in expression rates.
- The simulated trajectory contains a cell type elevated only in the treatment batch after the bifurcation point, verifiable by inspecting per-batch expression of gene A in the synthetic data.
- The synthetic counts mix reads from the reference cell populations in proportions consistent with the intended trajectory (Frankencell trajectory-mixing invariant).

## Limitations

- Authors caution that new batch-confounded or perturbation-induced cell states discovered on such simulated systems should be validated using orthogonal biological evidence to rule out under-correction of technical effects.
- Further improvements require better characterization of technical sources of bias; the simulation models batch of origin only as a proxy variable for hidden batch-specific factors.
- CODAL (and by extension this benchmark) does not remove the need for good experimental design and appropriate controls in real experiments.

## Evidence

- [methods] Frankencell generates synthetic differentiation trajectories by mixing reads from individual cells sampled from distinct, well-defined cell populations: "Frankencell generates synthetic differentiation trajectories by mixing reads from individual cells sampled from distinct, well-defined cell populations"
- [intro] batch of origin proxy: "the cells’ batch of origin can be used as a proxy variable to indicate which cells were subjected to a protocol using common reagents and under similar conditions."
- [discussion] control group guideline: "a simple guideline is to include a control group of cells that do not experience or are known not to respond to the perturbation in both treatment and control batches"
- [discussion] orthogonal validation caveat: "the existence of new batch-confounded or perturbation-induced cell states should be validated using orthogonal biological evidence to rule out the under-correction of technical effects"
- [readme] MIRA input format: [evidence span withheld — anchor status: external_doc]
