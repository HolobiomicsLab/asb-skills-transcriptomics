---
name: batch-effect-simulation
description: Use when benchmarking batch-effect removal, condition-associated key
  gene (CKG) detection, or perturbation prediction methods and ground-truth data with
  known condition effects and batch labels are required.
when_to_use_negative:
- Input is real experimental scRNA-seq data without ground truth — simulation is only
  for validation where true CKGs and condition effects are known
- Data lack multiple batches or multiple condition types, so no batch-condition structure
  can be simulated or disentangled
- Each batch is measured under only one condition, since there is insufficient information
  to disentangle batch and condition effects
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SymSim
  role: Simulator used to generate the nine multi-batch multi-condition count matrices
    with injected condition effects
- name: scDisInFact
  role: Target framework whose three tasks (batch effect removal, CKG detection, perturbation
    prediction) the simulated datasets validate; create_scdisinfact_dataset loads
    the simulated counts and metadata
  repo: github.com/ZhangLabGT/scDisInFact
- name: numpy
  role: Stores the 8 count matrices per dataset as numpy.array objects of shape (ncells,
    ngenes)
- name: pandas
  role: Builds the meta_cells dataframe with batch and condition columns aligned 1:1
    with the count matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45227-w
    title: 'scDisInFact: disentangled learning for integration and prediction of multi-batch
      multi-condition single-cell RNA-sequencing data'
  source_content_hash: sha256:4ea71f33bc466ad870efc19677456b97e4725bcd99f68d82bf484321b83b9b8d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SymSim
  - numpy
  - pandas
  - scDisInFact
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# batch-effect-simulation

## Summary

Generate ground-truth multi-batch multi-condition scRNA-seq datasets with SymSim by simulating a 2-batch × 2-condition-type factorial design (8 count matrices per dataset) and injecting uniform condition effects on condition-associated key genes (CKGs) at controlled perturbation strengths.

## When to use

Use when benchmarking batch-effect removal, condition-associated key gene (CKG) detection, or perturbation prediction methods and ground-truth data with known condition effects and batch labels are required.

## When NOT to use

- Input is real experimental scRNA-seq data without ground truth — simulation is only for validation where true CKGs and condition effects are known
- Data lack multiple batches or multiple condition types, so no batch-condition structure can be simulated or disentangled
- Each batch is measured under only one condition, since there is insufficient information to disentangle batch and condition effects

## Inputs

- SymSim simulation configuration for a 2-batch x 2-condition-type factorial design
- Parameter grid values: m_diff in {20, 50, 100} and perturbation parameter eps in {2, 4, 8}
- Designated condition-associated key gene (CKG) lists per grid point

## Outputs

- 9 simulated datasets, each with 8 numpy.array count matrices of shape (ncells, ngenes)
- Per-dataset meta_cells pandas.dataframe of shape (ncells, xx) with batch, condition-type, and cell-type columns
- Simulated datasets loadable via create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition 1", "condition 2"], batch_key="batch")

## How to apply

Configure SymSim with a 2-batch × 2-condition-type factorial design (treatment and disease severity), yielding 8 count matrices per simulated dataset. Enumerate a 9-dataset parameter grid crossing m_diff (number of condition-associated key genes) in {20, 50, 100} with the perturbation parameter eps in {2, 4, 8} to model different strengths of condition effect. For each grid point, simulate the count matrices with SymSim and inject uniform condition effects on the designated CKGs to encode the perturbation strength. Assemble a per-dataset metadata table with cell-level batch, condition-type, and cell-type labels whose rows align 1:1 with the simulated counts using pandas, and persist the 8 count matrices as numpy arrays so they can be loaded by create_scdisinfact_dataset with condition_key and batch_key columns.

## Related tools

- **SymSim** (Simulator used to generate the nine multi-batch multi-condition count matrices with injected condition effects)
- **scDisInFact** (Target framework whose three tasks (batch effect removal, CKG detection, perturbation prediction) the simulated datasets validate; create_scdisinfact_dataset loads the simulated counts and metadata) — github.com/ZhangLabGT/scDisInFact
- **numpy** (Stores the 8 count matrices per dataset as numpy.array objects of shape (ncells, ngenes))
- **pandas** (Builds the meta_cells dataframe with batch and condition columns aligned 1:1 with the count matrices)

## Examples

```
from scDisInFact import scdisinfact, create_scdisinfact_dataset; data_dict = create_scdisinfact_dataset(counts, meta_cells, condition_key=["condition 1", "condition 2"], batch_key="batch")
```

## Evaluation signals

- Each of the 9 datasets contains exactly 8 count matrices with matching shapes (ncells, ngenes)
- The meta_cells table includes batch and condition columns and its row count matches the counts matrices
- Row alignment between meta_cells and counts is 1:1 so create_scdisinfact_dataset accepts the data without errors
- Injected condition effects on CKGs scale with eps in {2, 4, 8}, producing distinguishable weak/medium/strong perturbation strengths
- Simulated ground truth (true CKGs, batch and condition labels) matches the m_diff grid {20, 50, 100} per dataset

## Limitations

- Simulated additivity may not reflect higher-order interactions: scDisInFact assumes condition and batch effects are additive in latent space, which limits modeling of higher-order effects
- Simulation encodes mutually independent condition types and condition effects independent of batch effects, which does not always hold in real-world datasets
- When each batch is measured under only one condition, batch and condition effects cannot be fully disentangled, so such simulation designs are uninformative for that task

## Evidence

- [methods] SimSim factorial design and parameter grid: [evidence span withheld — anchor status: not_found]
- [readme] Counts and metadata format: "the input count matrix `counts` (`numpy.array` of the shape `(ncells, ngenes)`) and meta data `meta_cells` (`pandas.dataframe` of the shape `(ncells, xx)`)"
- [readme] Metadata alignment requirement: "`meta_cells` must include columns corresponding to the batch ID and condition of each cells, and the cells in `meta_cells` should match `counts`"
- [methods] Loading simulated data into scDisInFact: "from scDisInFact import scdisinfact, create_scdisinfact_dataset"
- [abstract] Three validation tasks: "enabling it to simultaneously perform three tasks: batch effect removal, condition-associated key gene"
