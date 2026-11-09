---
name: wall-clock-runtime-profiling
description: Use when comparing computational efficiency of lineage reconstruction
  methods across dataset sizes, e.g. timing LinRace vs LinTIMaT on simulated barcode-expression
  data of 1024 and 2048 cells.
when_to_use_negative:
- The goal is accuracy evaluation (e.g. Robinson-Foulds or Nye similarity of inferred
  trees) rather than computational scalability — use tree-comparison workflows instead.
- Only a single cell count is available, so no runtime-versus-size trend can be established;
  the skill requires timing across a range of ncells.
- Methods being compared are not all runnable on the same character matrices (e.g.
  methods that cannot consume paired barcode-expression inputs), making wall-clock
  comparisons incommensurable.
tools:
- name: LinRace
  role: Timed integrated lineage reconstruction method run via NJ_asym_Dif (max_Iter=500,
    lambda1=10, lambda2=1)
  repo: https://github.com/ZhangLabGT/LinRace
- name: LinTIMaT
  role: Timed baseline integrated method that uses a combined likelihood function
    with a local search framework
- name: DCLEAR
  role: Timed distance-based baseline run via NJ_asym_DCLEAR (max_Iter=500, lambda1=10,
    lambda2=1)
- name: Cassiopeia
  role: Timed parsimony-based greedy solver run on the same character matrices
- name: TedSim
  role: Generates simulated CIFs and paired true counts at each cell count via SimulateCIFs
    and CIF2Truecounts
- name: ggplot2
  role: Plots running time versus number of cells per method
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:67fc66156a7a751f487d027caf40203e2be95e6716d45e81f345951342da4f45
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
  - TedSim
  - ggplot2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# wall-clock-runtime-profiling

## Summary

Benchmark the wall-clock running time of LinRace against LinTIMaT, DCLEAR, and Cassiopeia-greedy on simulated paired barcode–expression datasets of increasing cell count, reproducing Supplementary Fig. 9's scalability comparison of lineage reconstruction methods.

## When to use

Use when comparing computational efficiency of lineage reconstruction methods across dataset sizes, e.g. timing LinRace vs LinTIMaT on simulated barcode-expression data of 1024 and 2048 cells.

## When NOT to use

- The goal is accuracy evaluation (e.g. Robinson-Foulds or Nye similarity of inferred trees) rather than computational scalability — use tree-comparison workflows instead.
- Only a single cell count is available, so no runtime-versus-size trend can be established; the skill requires timing across a range of ncells.
- Methods being compared are not all runnable on the same character matrices (e.g. methods that cannot consume paired barcode-expression inputs), making wall-clock comparisons incommensurable.

## Inputs

- Simulated paired lineage barcode character matrices and gene expression counts (TedSim SimulateCIFs with N_char=16, ngenes=500) at ncells = 1024 and 2048
- kmeans-derived cell cluster states on top-50 PCs of the expression data
- 6-tip state tree '((t1:4,t2:4,t3:4,t4:4,t5:4,t6:4):2;)'

## Outputs

- runtime_vs_ncells.csv containing one wall-clock time column per method (linrace, LinTIMaT, cassiopeia, DCLEAR) indexed by cell count
- ggplot2 plot of running time versus number of cells per method (Supplementary Fig. 9 reproduction)

## How to apply

Simulate paired lineage and expression data with TedSim's SimulateCIFs (N_char=16, max_walk=8, cif_step=0.5, Sigma=0.5, p_a=0.8, mu in {0.025,0.05,0.075}) for ncells of 1024 and 2048 on the 6-tip state tree '((t1:4,t2:4,t3:4,t4:4,t5:4,t6:4):2;)', then generate paired true counts with CIF2Truecounts (ngenes=500, ncif=30, ge_prob=0.3). Wrap each solver call in proc.time(): run LinRace via NJ_asym_Dif on the transformed barcodes, kmeans-derived cluster states, and counts (max_Iter=500, lambda1=10, lambda2=1); run DCLEAR (NJ_asym_DCLEAR, max_Iter=500, lambda1=10, lambda2=1), the Cassiopeia-greedy parsimony solver, and LinTIMaT's combined-likelihood local search on the same inputs. Aggregate the recorded wall-clock times into runtime_vs_ncells.csv with one time column per method indexed by cell count, and plot running time versus number of cells per method with ggplot2.

## Related tools

- **LinRace** (Timed integrated lineage reconstruction method run via NJ_asym_Dif (max_Iter=500, lambda1=10, lambda2=1)) — https://github.com/ZhangLabGT/LinRace
- **LinTIMaT** (Timed baseline integrated method that uses a combined likelihood function with a local search framework)
- **DCLEAR** (Timed distance-based baseline run via NJ_asym_DCLEAR (max_Iter=500, lambda1=10, lambda2=1))
- **Cassiopeia** (Timed parsimony-based greedy solver run on the same character matrices)
- **TedSim** (Generates simulated CIFs and paired true counts at each cell count via SimulateCIFs and CIF2Truecounts)
- **ggplot2** (Plots running time versus number of cells per method)

## Examples

```
ptm <- proc.time(); tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF), cluster_id, expr_ZF, state_lineages, max_Iter = 500, lambda1 = 10, lambda2 = 1); total_time <- proc.time() - ptm
```

## Evaluation signals

- runtime_vs_ncells.csv exists and contains a time column for each of the four methods at every simulated cell count (1024 and 2048).
- LinRace's recorded runtime is lower than LinTIMaT's at the largest cell count (2048 cells), consistent with Supplementary Fig. 9.
- All solver runs use matched parameters (max_Iter=500, lambda1=10, lambda2=1) and identical character matrices, so times are comparable across methods.
- Recorded runtimes increase (or remain plausible) monotonically with cell count for each method; anomalous non-monotonic times indicate a timing or setup error.
- proc.time() difference captures only the solver call, not the simulation or I/O steps.

## Limitations

- Wall-clock times are machine-dependent, so results are reproducible only in relative (per-method ranking) terms on the same hardware.
- Simulated scalability is tested only up to 2048 cells with TedSim parameters (mu in {0.025,0.05,0.075}, N_char=16); larger systems may shift the runtime ordering.
- No software versions are stated for any R package, which can affect timing and behavior of the compared solvers.

## Evidence

- [methods] workflow timing anchor: "tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF),cluster_id,expr_ZF,state_lineages,max_Iter = 500)"
- [methods] simulation setup: "cifs <- SimulateCIFs(ncells,phyla,p_a = p_a,n_CIF = n_cif,n_diff = n_diff,step = cif_step,p_d = p_d, Sigma = 0.5,mu = mu, N_char = N_char, max_walk = max_walk"
- [methods] paired count generation: "true_counts_res <- CIF2Truecounts(ngenes = 500,ncif = n_cif,ge_prob = 0.3,ncells = ncells, cif_res = cif_res)"
- [intro] baseline method descriptions: "LinTIMaT22 develops a combined likelihood function and uses a local search framework to search for the tree with the maximum likelihood"
- [readme] README install/run context: [evidence span withheld — anchor status: not_found]
