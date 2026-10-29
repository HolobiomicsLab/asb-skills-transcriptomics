---
name: crispr-barcode-simulation
description: Use when a lineage-reconstruction benchmark needs paired barcode-and-expression
  data with a known true tree, e.g., sweeping mutation rates on the C. elegans L1
  tree or TedSim-simulated phyla.
when_to_use_negative:
- You have real, experimentally acquired lineage barcode data with a validated tree
  — simulation is only a benchmark surrogate
- The system has reversible cell-state transitions or synchronous differentiation,
  which LinRace/TedSim-based asymmetric-division assumptions do not model
- You need barcode technology realism beyond TedSim's CRISPR mutation/dropout model
  (e.g., high-diversity recording outcomes not parameterized by mu and N_char)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: TedSim
  role: Simulates cell identity fields and CRISPR barcode mutations on a known tree
    via SimulateCIFs; also generates true expression counts via CIF2Truecounts
  repo: https://github.com/ZhangLabGT/LinRace
- name: LinRace
  role: Downstream method applied to simulated barcodes plus expression (NJ_asym_Dif,
    max_Iter = 500) to test simulation-benchmark discrimination
  repo: https://github.com/ZhangLabGT/LinRace
- name: phangorn
  role: Encodes simulated barcodes as phyDat unique allele states and builds the barcode-only
    NJ baseline with dist.hamming + nj
- name: TreeDist
  role: Scores simulated-data reconstructions with normalized NyeSimilarity against
    the true tree
- name: ape
  role: Computes normalized Robinson-Foulds distance (RF.dist, normalize = TRUE) of
    reconstructed trees against the simulated ground-truth tree
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:2ff2f9603d7ab4a45b37e021520bf355a06878ceb7505956af319a2ca8e07905
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/LinRace
  tools:
  - Cassiopeia
  - DCLEAR
  - LinRace
  - LinTIMaT
  - R
  - Startle
  - TedSim
  - TreeDist
  - ape
  - ggplot2
  - phangorn
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6086956521739131
n_claims: 23
---

# crispr-barcode-simulation

## Summary

Simulate CRISPR lineage barcodes on a known ground-truth lineage tree with TedSim's SimulateCIFs, optionally with dropouts, to create paired barcode/expression benchmarks for evaluating lineage-reconstruction methods such as LinRace.

## When to use

Use when a lineage-reconstruction benchmark needs paired barcode-and-expression data with a known true tree, e.g., sweeping mutation rates on the C. elegans L1 tree or TedSim-simulated phyla.

## When NOT to use

- You have real, experimentally acquired lineage barcode data with a validated tree — simulation is only a benchmark surrogate
- The system has reversible cell-state transitions or synchronous differentiation, which LinRace/TedSim-based asymmetric-division assumptions do not model
- You need barcode technology realism beyond TedSim's CRISPR mutation/dropout model (e.g., high-diversity recording outcomes not parameterized by mu and N_char)

## Inputs

- Ground-truth lineage tree (Newick), e.g., the C. elegans L1 tree from CeLaVi
- Simulation parameters: N_char (target sites), p_a, n_CIF, Sigma, max_walk, mu (mutation rate), p_d (dropout probability), ncells, replicates
- Optional measured scRNA-seq expression matrix (e.g., 363 cells x 93 genes C. elegans L1 matrix from ref. 35)

## Outputs

- Simulated lineage barcode character matrices (per cell allele states) per mu/ncells/dropout condition and replicate
- Simulated cell identity fields (CIFs) and, optionally, true expression counts via CIF2Truecounts
- Ground-truth trees retained for RF-distance, Nye similarity, and CID scoring (e.g., tabulated in fig3c_l1_benchmark.csv)

## How to apply

Load a ground-truth tree (e.g., the C. elegans L1 Newick from CeLaVi) and call TedSim's SimulateCIFs with parameters such as N_char = 16 target sites, p_a = 0.8, n_CIF = 30, Sigma = 0.5, max_walk = 8, sweeping mutation rates mu in c(0.025, 0.05, 0.075) with ncells in c(1024, 2048), 5 replicates per condition, under both dropout-free (p_d = 0) and dropout conditions. Downstream, simulated true expression counts can be generated with CIF2Truecounts (e.g., ngenes = 500, ge_prob = 0.3). Pair the simulated barcodes with real measured expression (e.g., the 363 cells x 93 genes C. elegans L1 matrix) or with simulated expression to build benchmark conditions. Encode barcodes as character matrices (phyDat unique allele states) so that baselines like Hamming-distance NJ and competitors (DCLEAR, Cassiopeia, LinTIMaT, Startle) run on identical inputs. Keep the true tree from the simulation as the reference for scoring.

## Related tools

- **TedSim** (Simulates cell identity fields and CRISPR barcode mutations on a known tree via SimulateCIFs; also generates true expression counts via CIF2Truecounts) — https://github.com/ZhangLabGT/LinRace
- **LinRace** (Downstream method applied to simulated barcodes plus expression (NJ_asym_Dif, max_Iter = 500) to test simulation-benchmark discrimination) — https://github.com/ZhangLabGT/LinRace
- **phangorn** (Encodes simulated barcodes as phyDat unique allele states and builds the barcode-only NJ baseline with dist.hamming + nj)
- **TreeDist** (Scores simulated-data reconstructions with normalized NyeSimilarity against the true tree)
- **ape** (Computes normalized Robinson-Foulds distance (RF.dist, normalize = TRUE) of reconstructed trees against the simulated ground-truth tree)

## Examples

```
cifs <- SimulateCIFs(ncells = 1024, phyla, p_a = 0.8, n_CIF = 30, p_d = 0, Sigma = 0.5, mu = 0.05, N_char = 16, max_walk = 8)
```

## Evaluation signals

- Simulated conditions reproduce across 5 replicates per mu x ncells x dropout condition without stochastic failures (no empty or degenerate barcode matrices)
- Reconstructions on simulated barcodes yield finite normalized RF.dist, NyeSimilarity, and CID scores against the retained true tree
- Benchmark outcome matches the deposited Fig. 3c Source Data values, with LinRace achieving the lowest RF distance across swept mutation-rate and dropout conditions
- Dropout (p_d > 0) and dropout-free (p_d = 0) runs produce separable result sets consistent with Fig. 3c vs. Supplementary Fig. 10
- Barcode-only NJ baseline (dist.hamming + nj on phyDat-encoded unique allele states, multi2di with uniform edge lengths) completes and serves as the expected lower bound

## Limitations

- Benchmark barcodes are simulated (TedSim), so real-data CRISPR recording artifacts beyond the mu/p_d parameterization are not captured
- LinRace assumes irreversible cell-state transitions and that all developmental states exist in current-day cells; simulated scenarios violating these assumptions may flatter or penalize methods unfairly
- Results depend on lineage barcode technology quality (high diversity and coverage) that current workflows may not achieve

## Evidence

- [methods] simulation of cell identity fields and mutations with TedSim (SimulateCIFs): [evidence span withheld — anchor status: not_found]
- [methods] SimulateCIFs call with parameters: "cifs <- SimulateCIFs(ncells,phyla,p_a = p_a,n_CIF = n_cif,n_diff = n_diff,step = cif_step,p_d = p_d, Sigma = 0.5,mu = mu, N_char = N_char, max_walk = max_walk"
- [methods] true expression counts from CIFs: "true_counts_res <- CIF2Truecounts(ngenes = 500,ncif = n_cif,ge_prob = 0.3,ncells = ncells, cif_res = cif_res)"
- [methods] NJ baseline on Hamming distance: "dist_h <- dist.hamming(sim_data); Treenj = nj(dist_h)"
- [readme] README install instructions: [evidence span withheld — anchor status: external_doc]
