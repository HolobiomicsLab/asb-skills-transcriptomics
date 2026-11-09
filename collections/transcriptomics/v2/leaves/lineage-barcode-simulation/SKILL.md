---
name: lineage-barcode-simulation
description: Use when benchmarking or validating lineage tree reconstruction methods
  and you need simulated ground-truth trees, leaf barcodes (muts_leaves), and paired
  expression counts over a grid of mutation rates, dropouts, and cell counts.
when_to_use_negative:
- You already have real paired lineage-barcode/scRNA-seq data (e.g., zebrafish scGESTALT
  cm_ZF1_F3 matrix) and need reconstruction rather than simulation
- The downstream method cannot consume TedSim's mutation-matrix format and no MutsTransform-compatible
  preprocessing exists
- The study requires reversible cell state transitions, which TedSim's asymmetric-division
  model and LinRace do not model
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3173
tools:
- name: TedSim
  role: Simulates cell identity fields, mutations, and true counts (SimulateCIFs,
    CIF2Truecounts) to produce ground-truth trees, barcodes, and expression
  repo: https://github.com/ZhangLabGT/LinRace
- name: LinRace
  role: Consumes the simulated paired data (after MutsTransform and Preprocessing)
    for lineage reconstruction and benchmarking
  repo: https://github.com/ZhangLabGT/LinRace
- name: R
  role: Environment in which TedSim and LinRace simulation and preprocessing functions
    are executed
provenance:
  source_task_ids:
  - task_006
  - task_003
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:328a74703a89de629db793f23d009bf4106518963bea85283290050fafe1e96f
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
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5652173913043478
n_claims: 23
---

# lineage-barcode-simulation

## Summary

Simulate paired lineage-barcode and gene-expression single-cell datasets with TedSim (SimulateCIFs plus CIF2Truecounts) to generate ground-truth cell division trees, leaf barcode matrices, and true expression counts for benchmarking lineage reconstruction methods like LinRace.

## When to use

Use when benchmarking or validating lineage tree reconstruction methods and you need simulated ground-truth trees, leaf barcodes (muts_leaves), and paired expression counts over a grid of mutation rates, dropouts, and cell counts.

## When NOT to use

- You already have real paired lineage-barcode/scRNA-seq data (e.g., zebrafish scGESTALT cm_ZF1_F3 matrix) and need reconstruction rather than simulation
- The downstream method cannot consume TedSim's mutation-matrix format and no MutsTransform-compatible preprocessing exists
- The study requires reversible cell state transitions, which TedSim's asymmetric-division model and LinRace do not model

## Inputs

- A predefined state/clone tree (e.g., 6-terminal-state tree string) passed as the phyla argument
- Simulation parameter grid: ncells, N_char, mu (mutation rate), p_d (dropout probability), p_a, n_CIF, n_diff, cif_step, Sigma, max_walk
- Output of SimulateCIFs (list including tree_ct, muts_leaves, CIF states)
- CIF2Truecounts parameters: ngenes=500, ncif=30, ge_prob=0.3

## Outputs

- Ground-truth cell division tree (tree_ct) with cell_* tip labels
- Leaf-cell barcode character matrix (muts_leaves)
- True gene expression count matrix (ngenes=500) via CIF2Truecounts
- Cell state labels and state_lineages for preprocessing (cluster_kmeans mapping)
- Collected metric tables such as qr_qh_vs_mutation_rate.csv or benchmark results CSVs

## How to apply

Run TedSim's SimulateCIFs on a predefined state tree (e.g., the 6-terminal-state tree '((t1:4,t2:4,t3:4,t4:4,t5:4,t6:4):2)') with parameters such as N_char=16 barcode characters, max_walk=8, n_CIF=30, n_diff=20, cif_step=0.5, Sigma=0.5, p_a=0.8, and a grid of mutation rates (e.g., mu in {0.025, 0.05, 0.075, 0.4}) and cell counts (1024, 2048, 4096), with p_d=0 for no-dropout and non-zero p_d for dropout conditions. Extract the ground-truth cell division tree (tree_ct with cell_* tip labels), the leaf mutation matrix (muts_leaves, first ncells rows), and cell state labels from the simulation output. Then generate paired expression with CIF2Truecounts(ngenes=500, ge_prob=0.3). Build cell-state intermediate labels and state_lineages via the Preprocessing mapping and transform barcodes with MutsTransform before downstream reconstruction. Derived quantities such as reconstruction potential (Qr, Eq. 13) and homoplasy edge ratio (Qh, Eq. 14) can be computed from muts_leaves and tree_ct to characterize barcode quality across the parameter grid.

## Related tools

- **TedSim** (Simulates cell identity fields, mutations, and true counts (SimulateCIFs, CIF2Truecounts) to produce ground-truth trees, barcodes, and expression) — https://github.com/ZhangLabGT/LinRace
- **LinRace** (Consumes the simulated paired data (after MutsTransform and Preprocessing) for lineage reconstruction and benchmarking) — https://github.com/ZhangLabGT/LinRace
- **R** (Environment in which TedSim and LinRace simulation and preprocessing functions are executed)

## Examples

```
cifs <- SimulateCIFs(ncells=1024, phyla, p_a=0.8, n_CIF=30, n_diff=20, step=0.5, p_d=0, Sigma=0.5, mu=0.05, N_char=16, max_walk=8, unif_on=FALSE); true_counts_res <- CIF2Truecounts(ngenes=500, ncif=30, ge_prob=0.3, ncells=1024, cif_res=cif_res)
```

## Evaluation signals

- The simulation output list contains tree_ct with cell_* tip labels and muts_leaves whose first ncells rows match the simulated cells
- The homoplasy edge ratio Qh reaches approximately 0.75 on 1024-cell, 16-character datasets when dropouts are present, varying with mutation rate similarly to reconstruction method performance
- Downstream NJ on Hamming distance (dist.hamming then nj with multi2di) yields a tree whose normalized RF.dist against tree_ct can be computed without errors
- CIF2Truecounts returns a 500-gene true count matrix over the simulated cells, and kmeans clustering on its top-50 PCs maps cells to states consistent with the known state tree
- The written CSV (e.g., qr_qh_vs_mutation_rate.csv) contains Qr/Qh values for every mutation-rate x dropout-setting combination in the sweep

## Limitations

- LinRace (and the simulation's asymmetric division model) assumes irreversible cell state transitions; reversible transitions are not modeled
- All cell states during development are assumed to exist in current-day cells; ancestral states are restricted to states present in the scRNA-seq data
- Barcode data quality depends on short barcode length, dropout events, and biased/non-optimal mutation rates, which must be swept explicitly to assess sensitivity
- Software versions for the R packages are not stated in the article

## Evidence

- [methods] simulation of cell identity fields and mutations with TedSim (SimulateCIFs): "cifs <- SimulateCIFs(ncells,phyla,p_a = p_a,n_CIF = n_cif,n_diff = n_diff,step = cif_step,p_d = p_d, Sigma = 0.5,mu = mu, N_char = N_char, max_walk = max_walk, SIF_res = returnlist, unif_on = FALSE)"
- [methods] simulation of true gene expression counts: "true_counts_res <- CIF2Truecounts(ngenes = 500,ncif = n_cif,ge_prob = 0.3,ncells = ncells, cif_res = cif_res)"
- [methods] neighbor joining on Hamming distance of barcode data: "dist_h <- dist.hamming(sim_data); Treenj = nj(dist_h)"
- [readme] README installation instructions for LinRace/TedSim ecosystem: "The R package for LinRace can be installed as:
`devtools::install_github("https://github.com/ZhangLabGT/LinRace")`"
- [abstract] abstract description of LinRace using asymmetric cell division model: "which integrates lineage barcode and gene expression data using asymmetric cell division model and infers cell lineages and ancestral cell states using Neighbor-Joining and maximum-likelihood"
