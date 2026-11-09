---
name: bifurcation-simulation-data-validation
description: Use when validating that MuTrans TCS distinguishes stable from transition
  cells on the saddle-node bifurcation simulation, or when reproducing the Fig. 2a
  sharp-transition ordering from example_saddle_node.mlx.
when_to_use_negative:
- Input is experimental single-cell data rather than the simulation; other MuTrans
  example workflows (EMT, iPSC, myelopoiesis) apply instead
- Dataset is too small — the paper notes a sufficiently large number of cells is required
  for high-resolution identification of transition cells
- Goal is lineage or landscape reconstruction rather than TCS-based stable/transition
  cell validation (use InferLineage or ConstructLandscape skills)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MuTrans
  role: 'Multiscale stochastic dynamics framework: rwTPM construction, DynamicalAnalysis
    coarse-graining, and TCS computation via Transcendental/GeneAnalysis'
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: Transcendental
  role: Transition cell analysis procedure that separates transition cells from stable
    cells and computes tau_ij(x) for each cell
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: transition cell score (TCS)
  role: Score measuring relative position of cell x in different clusters; the output
    validated against stable/transition cell ground truth in the simulation
  repo: https://github.com/cliffzhou92/MuTrans-release
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:eafcd6520b0df2370cb452dd09b8b98fc5601e370482332a91756faa82b64178
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - MuTrans
  - Transcendental
  - transition cell score (TCS)
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bifurcation-simulation-data-validation

## Summary

Reproduces and validates MuTrans's transition cell score (TCS) on the simulated stochastic saddle-node bifurcation dataset (N=2,000 cells generated via Euler-Maruyama solution of an over-damped Langevin equation), verifying that stable-branch cells receive TCS near 0 or 1 while transition-layer cells occupy intermediate values in (0,1).

## When to use

Use when validating that MuTrans TCS distinguishes stable from transition cells on the saddle-node bifurcation simulation, or when reproducing the Fig. 2a sharp-transition ordering from example_saddle_node.mlx.

## When NOT to use

- Input is experimental single-cell data rather than the simulation; other MuTrans example workflows (EMT, iPSC, myelopoiesis) apply instead
- Dataset is too small — the paper notes a sufficiently large number of cells is required for high-resolution identification of transition cells
- Goal is lineage or landscape reconstruction rather than TCS-based stable/transition cell validation (use InferLineage or ConstructLandscape skills)

## Inputs

- Saddle-node bifurcation simulation data (Euler-Maruyama solution of over-damped Langevin equations, N=2,000 cells) from MuTrans-release Example/Data deposit
- Reference TCS output from example_saddle_node.mlx tutorial notebook
- K_cluster (number of attractors) parameter setting from the tutorial

## Outputs

- tcs_saddle_node.csv with per-cell cell ID, membership fractions, and TCS values
- Attractor basin assignments, coarse-grained transition probabilities, and soft membership function rho*(x) from DynamicalAnalysis
- Validation report of fraction of stable cells with TCS < 0.05 or > 0.95

## How to apply

Load the saddle-node bifurcation simulation (Euler-Maruyama solution of the over-damped Langevin equation, N=2,000 cells) from the MuTrans-release Data deposit, then run pm.dynamical_analysis / DynamicalAnalysis on the random-walk transition probability matrix (rwTPM) with K_cluster set per the tutorial example_saddle_node.mlx to obtain attractor basins, coarse-grained transition probabilities, and the soft membership function rho*(x). Execute the Transcendental procedure via GeneAnalysis between the starting and targeting states to compute tau_ij(x) = rho_i*(x)/(rho_i*(x)+rho_j*(x)) for each cell. Verify that stable-branch cells show TCS approaching one or zero (meta-stability), while transition-layer cells form a continuum of intermediate values, matching the reported Fig. 2a sharp-transition pattern. Write per-cell results (cell ID, membership fractions, TCS) to tcs_saddle_node.csv and compare against the tutorial notebook's reference TCS output within tolerance.

## Related tools

- **MuTrans** (Multiscale stochastic dynamics framework: rwTPM construction, DynamicalAnalysis coarse-graining, and TCS computation via Transcendental/GeneAnalysis) — https://github.com/cliffzhou92/MuTrans-release
- **Transcendental** (Transition cell analysis procedure that separates transition cells from stable cells and computes tau_ij(x) for each cell) — https://github.com/cliffzhou92/MuTrans-release
- **transition cell score (TCS)** (Score measuring relative position of cell x in different clusters; the output validated against stable/transition cell ground truth in the simulation) — https://github.com/cliffzhou92/MuTrans-release

## Examples

```
par = {"K_cluster": 2.0}; adata = pm.dynamical_analysis(adata, par); Genes = pm.gene_analysis(adata, si=0, sf=1)
```

## Evaluation signals

- Cells fluctuating around the two stable branches receive TCS values approaching one or zero respectively (meta-stability check)
- Fraction of stable cells with TCS < 0.05 or > 0.95 is consistent with the reported Fig. 2a pattern
- Computed TCS values match example_saddle_node.mlx reference TCS output within tolerance (output_matches_reference)
- Transition-layer cells form a continuum of intermediate TCS values in (0,1), reproducing the sharp-transition ordering
- Attractor basins and soft membership rho*(x) returned by DynamicalAnalysis correspond to the two stable branches of the saddle-node bifurcation

## Limitations

- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the method
- Continuum limit of the soft membership function in embedded over-damped Langevin dynamical systems remains an open problem
- Matlab requires the optimization toolbox (and Curve Fitting toolbox for Transcendental's 'smooth' function); tested in Matlab R2019b and R2020a

## Evidence

- [methods] Euler-Maruyama simulation and TCS validation: "The simulation data was generated by the Euler-Maruyama method to solve the overdamped Langevin equations"
- [methods] TCS definition and purpose: [evidence span withheld — anchor status: not_found]
- [methods] Transcendental procedure: "MuTrans performs the Transcendental (transition cells and relevant analysis) procedure on each transition process to identify the transition cells from the stable cells"
- [intro] TCS distinguishes attractors from transition cells: "the Transition Cell Score (TCS) and transition entropy allows us to distinguish between attractors and transition cells"
- [readme] Saddle-node example notebook: "Saddle-Node Bifurcation | Simulation Data in this study | [example_saddle_node.mlx](https://github.com/cliffzhou92/MuTrans-release/blob/main/Example/example_saddle_node.mlx)"
- [readme] DynamicalAnalysis outputs: "%% Run the multi-scale analysis, returns the attractor basins, coarse-grained transition probabilities and membership assignment"
