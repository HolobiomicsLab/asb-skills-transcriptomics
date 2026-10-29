---
name: r-package-workflow-scripting
description: Use when reproducing or extending an R-based pipeline that installs Bioconductor/GitHub
  packages via devtools, simulates lineage data, calls method functions like NJ_asym_Dif,
  and evaluates phylogenetic trees against a ground truth.
when_to_use_negative:
- The analysis is entirely within a Python ecosystem (e.g. DCLEAR or Cassiopeia runs)
  and no R-package interop is needed — invoke those tools natively instead of wrapping
  them in R
- Input is a pre-computed benchmark score table that only needs tabulation or plotting
  — simple R scripting on the CSV suffices without the LinRace/TedSim package stack
- The data are unpaired (barcode-only or expression-only); LinRace's NJ_asym_Dif requires
  paired barcode and expression inputs
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LinRace
  role: R package providing NJ_asym_Dif, Preprocessing, MutsTransform and evaluation
    utilities; installed via devtools from GitHub
  repo: https://github.com/ZhangLabGT/LinRace
- name: TedSim
  role: R package for simulating paired CIF/barcode data (SimulateCIFs) and true counts
    (CIF2Truecounts)
- name: phangorn
  role: dist.hamming and nj for the barcode-only neighbor-joining baseline
- name: ape
  role: RF.dist for normalized Robinson-Foulds scoring against ground-truth trees
- name: TreeDist
  role: NyeSimilarity(normalize = TRUE) for pairwise tree comparison
- name: umap
  role: UMAP embedding of top-50 PCs for visualization and trajectory input
- name: ggplot2
  role: Plotting of reconstructed trees and benchmark results
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:ad160859f686aefc7944b10f39639f37a888de558b36e6985af49b684fd037f2
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
  - umap
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# r-package-workflow-scripting

## Summary

An R-package-based scripting workflow pattern for cell lineage reconstruction benchmarking, chaining GitHub-installed R packages (LinRace, TedSim) with ape/phangorn/TreeDist to simulate paired barcode/expression data, build trees, and score them with RF distance, Nye similarity, and CID.

## When to use

Use when reproducing or extending an R-based pipeline that installs Bioconductor/GitHub packages via devtools, simulates lineage data, calls method functions like NJ_asym_Dif, and evaluates phylogenetic trees against a ground truth.

## When NOT to use

- The analysis is entirely within a Python ecosystem (e.g. DCLEAR or Cassiopeia runs) and no R-package interop is needed — invoke those tools natively instead of wrapping them in R
- Input is a pre-computed benchmark score table that only needs tabulation or plotting — simple R scripting on the CSV suffices without the LinRace/TedSim package stack
- The data are unpaired (barcode-only or expression-only); LinRace's NJ_asym_Dif requires paired barcode and expression inputs

## Inputs

- R session with devtools to install ZhangLabGT/LinRace from GitHub
- TedSim simulation parameters (phyla tree e.g. (t1:4,...,t6:4):2, N_char, Sigma, mu grid, ncells grid)
- Barcode character matrix (e.g. cm_ZF1_F3.txt read via read.csv with sep='\t', row.names=1)
- Gene expression count matrix (e.g. zebrafish scGESTALT ZF1_F3_DrImpute counts from GSE117542)
- Ground-truth cell division tree for RF distance scoring

## Outputs

- Inferred phylogenetic trees (phylo objects) from LinRace and baseline methods
- Normalized RF distance and Nye similarity matrices / scores per method and condition
- Benchmark results CSV aggregating RF distance, Nye similarity, and CID per method per simulation condition
- R Markdown vignette outputs (html_vignette) documenting the workflow

## How to apply

Install the R package from GitHub with devtools::install_github('https://github.com/ZhangLabGT/LinRace') following the README, then load dependencies in the article's vignette style (library(LinRace), library(TedSim), plus ape, phangorn, TreeDist, umap, ggplot2). Follow the simulated-data vignette structure: SimulateCIFs on a phyla tree with parameters like N_char = 16, Sigma = 0.5, then CIF2Truecounts(ngenes = 500, ge_prob = 0.3) to generate counts; preprocess with Preprocessing and MutsTransform before tree building. Run the method call NJ_asym_Dif(muts_leaves, cluster_id, expr, state_lineages, max_Iter = 500) and baselines such as dist.hamming + nj for barcode-only neighbor joining. Score trees with RF.dist(normalize = TRUE) against the ground-truth tree and pairwise NyeSimilarity(normalize = TRUE) via TreeDist, aggregating into a benchmark CSV. Keep parameters (mu grids, ncells lists, max_Iter) aligned with the source card workflow and validate outputs against the paper's Fig. 2 Source Data reference.

## Related tools

- **LinRace** (R package providing NJ_asym_Dif, Preprocessing, MutsTransform and evaluation utilities; installed via devtools from GitHub) — https://github.com/ZhangLabGT/LinRace
- **TedSim** (R package for simulating paired CIF/barcode data (SimulateCIFs) and true counts (CIF2Truecounts))
- **phangorn** (dist.hamming and nj for the barcode-only neighbor-joining baseline)
- **ape** (RF.dist for normalized Robinson-Foulds scoring against ground-truth trees)
- **TreeDist** (NyeSimilarity(normalize = TRUE) for pairwise tree comparison)
- **umap** (UMAP embedding of top-50 PCs for visualization and trajectory input)
- **ggplot2** (Plotting of reconstructed trees and benchmark results)

## Examples

```
tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF), cluster_id, expr_ZF, state_lineages, max_Iter = 500); print(RF.dist(tree_linrace, tree_ct, normalize = TRUE))
```

## Evaluation signals

- devtools::install_github of ZhangLabGT/LinRace completes without error and library(LinRace) loads in a fresh R session
- Vignette reproduces: running the LinRace_test.Rmd simulated-data vignette yields trees that can be scored with RF.dist and NyeSimilarity without modification
- kmeans clustering (centers = 20 on top-50 PCs for zebrafish data) and state_lineages mapping produce non-empty cluster labels and valid state_lineages input to NJ_asym_Dif
- Normalized RF.dist values against the ground-truth tree fall in [0,1] and reproduce the ordering in Fig. 2 Source Data (LinRace-TST/IST best under dropout at 1024 cells)
- Benchmark CSV contains one row per method × condition (mu, ncells, dropout) matching the defined grid

## Limitations

- Software versions are not stated for any of the R packages in the methods ('library(LinRace)' only), so exact reproducibility may require pinning versions yourself
- Raw matrix.mtx.gz files for the zebrafish vignette were not uploaded due to GitHub LFS quota; data must be fetched from GEO accession GSE117542
- LinRace assumes irreversible cell state transitions and that all developmental cell states exist in current-day cells, so the R workflow is best suited to asynchronously differentiating systems

## Evidence

- [readme] README installation snippet: [evidence span withheld — anchor status: external_doc]
- [methods] PCA workflow step: "pca_res <- prcomp(expr_ZF,scale. = TRUE)"
- [methods] clustering workflow step: "cluster_id <- kmeans(pca_res$x[,1:50], centers = 20)$cluster"
- [methods] tree building call: "tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF),cluster_id,expr_ZF,state_lineages,max_Iter = 500)"
- [methods] tree comparison step: "Nye_mutual[i,j] <- NyeSimilarity(tree_list[[i]], tree_list[[j]], normalize = TRUE)"
- [methods] RF distance evaluation: "print( RF.dist(TreeNJ_h, tree_ct, normalize = TRUE) )"
