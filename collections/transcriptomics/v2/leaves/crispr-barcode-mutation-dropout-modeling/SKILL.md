---
name: crispr-barcode-mutation-dropout-modeling
description: Use when you need simulated ground-truth cell division trees with paired
  barcode/expression data under varying mutation rates, dropout conditions, and cell
  counts (e.g., 1024–4096 cells) to benchmark lineage tree inference methods.
when_to_use_negative:
- Analyzing real lineage-barcode data with no ground-truth division tree — RF distance
  and CID against truth cannot be computed; use Nye similarity among methods or biological
  validation instead
- The biological system has reversible cell state transitions — LinRace assumes irreversible
  state transitions, so simulated asymmetric-division benchmarks would not represent
  that regime
- Ancestral/intermediate cell states are fast-changing and absent from current-day
  scRNA-seq — the state-label reconstruction step will be misspecified
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3293
tools:
- name: LinRace
  role: Integrates barcode and expression data via the asymmetric division model (NJ_asym_Dif,
    max_Iter = 500, lambda1 = 10, lambda2 = 1); also provides Preprocessing, MutsTransform,
    and AncesInfer
  repo: https://github.com/ZhangLabGT/LinRace
- name: TedSim
  role: Simulates cell identity fields, barcode mutations (SimulateCIFs), and true
    gene expression counts (CIF2Truecounts) with controlled mutation rates and dropouts
- name: DCLEAR
  role: Distance-based baseline lineage reconstruction method compared under identical
    simulated inputs
- name: Cassiopeia
  role: Parsimony-based baseline tree builder on the same barcode matrices
- name: Startle
  role: Baseline enforcing non-modifiability of CRISPR/Cas9 mutations, run with NNI
- name: LinTIMaT
  role: Maximum-likelihood baseline using a combined likelihood function and local
    search
- name: phangorn
  role: Computes dist.hamming barcode distances and neighbor-joining baseline trees
    (nj, multi2di) plus normalized RF.dist scoring
- name: TreeDist
  role: Computes normalized pairwise NyeSimilarity among inferred method trees
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:b6475f3a9316227fffd3440d46fd6e198692fa310d78d39f240233b4002a6cb8
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
  - phangorn
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7083333333333334
n_claims: 24
---

# crispr-barcode-mutation-dropout-modeling

## Summary

Simulates paired CRISPR lineage-barcode and scRNA-seq datasets with TedSim under controlled mutation rates (e.g., mu = 0.025–0.075, 16 barcode characters, Sigma = 0.5), with and without dropout events, to benchmark lineage reconstruction methods like LinRace against Cassiopeia, DCLEAR, Startle, and LinTIMaT using RF distance, Nye similarity, and CID.

## When to use

Use when you need simulated ground-truth cell division trees with paired barcode/expression data under varying mutation rates, dropout conditions, and cell counts (e.g., 1024–4096 cells) to benchmark lineage tree inference methods.

## When NOT to use

- Analyzing real lineage-barcode data with no ground-truth division tree — RF distance and CID against truth cannot be computed; use Nye similarity among methods or biological validation instead
- The biological system has reversible cell state transitions — LinRace assumes irreversible state transitions, so simulated asymmetric-division benchmarks would not represent that regime
- Ancestral/intermediate cell states are fast-changing and absent from current-day scRNA-seq — the state-label reconstruction step will be misspecified

## Inputs

- Known cell division tree topology (e.g., TedSim 6-terminal-state phyla tree)
- Simulation parameters: N_char = 16 barcode characters, Sigma = 0.5, mu in {0.025, 0.05, 0.075}, ncells in {1024, 2048, 4096}, dropout on/off
- Barcode character matrix (muts_leaves) and gene expression count matrix (500 genes, ge_prob = 0.3)
- Cell cluster labels and state_lineages from Preprocessing

## Outputs

- Simulated paired barcode/expression datasets with ground-truth cell division trees
- Inferred lineage trees from LinRace (TST/IST) and baseline methods
- Benchmark results CSV with normalized RF.dist, Nye similarity (TreeDist), and CID per method and condition

## How to apply

Simulate cell identity fields and barcode mutations with TedSim's SimulateCIFs on a known division tree (e.g., the 6-terminal-state tree (t1:4,t2:4,t3:4,t4:4,t5:4,t6:4):2), using N_char = 16, Sigma = 0.5, and a grid of mutation rates (mu_list = c(0.025, 0.05, 0.075)) and cell counts (ncells_list = c(1024, 2048)), both with and without dropouts. Generate true expression counts with CIF2Truecounts (ngenes = 500, ge_prob = 0.3). Build cell-state intermediate labels and state_lineages through LinRace's Preprocessing mapping (kmeans clustering + depth to unique states) and transform barcodes with MutsTransform. Run LinRace's NJ_asym_Dif (max_Iter = 500, lambda1 = 10, lambda2 = 1) plus baselines — neighbor joining on barcode Hamming distance (dist.hamming then nj with multi2di), DCLEAR, Cassiopeia, Startle (NNI), and LinTIMaT — on identical inputs. Score each inferred tree against the ground truth with normalized RF.dist, compute pairwise normalized NyeSimilarity via TreeDist, and aggregate per method/condition into a benchmark CSV validated against the paper's Fig. 2 Source Data.

## Related tools

- **LinRace** (Integrates barcode and expression data via the asymmetric division model (NJ_asym_Dif, max_Iter = 500, lambda1 = 10, lambda2 = 1); also provides Preprocessing, MutsTransform, and AncesInfer) — https://github.com/ZhangLabGT/LinRace
- **TedSim** (Simulates cell identity fields, barcode mutations (SimulateCIFs), and true gene expression counts (CIF2Truecounts) with controlled mutation rates and dropouts)
- **DCLEAR** (Distance-based baseline lineage reconstruction method compared under identical simulated inputs)
- **Cassiopeia** (Parsimony-based baseline tree builder on the same barcode matrices)
- **Startle** (Baseline enforcing non-modifiability of CRISPR/Cas9 mutations, run with NNI)
- **LinTIMaT** (Maximum-likelihood baseline using a combined likelihood function and local search)
- **phangorn** (Computes dist.hamming barcode distances and neighbor-joining baseline trees (nj, multi2di) plus normalized RF.dist scoring)
- **TreeDist** (Computes normalized pairwise NyeSimilarity among inferred method trees)

## Examples

```
cifs <- SimulateCIFs(ncells, phyla, p_a = p_a, n_CIF = n_cif, n_diff = n_diff, Sigma = 0.5, mu = mu, N_char = N_char, SIF_res = returnlist, unif_on = FALSE); true_counts_res <- CIF2Truecounts(ngenes = 500, ge_prob = 0.3, cif_res = cifs)
```

## Evaluation signals

- Normalized RF.dist of each inferred tree against the TedSim ground-truth cell division tree is computed and comparable across methods
- Pairwise normalized NyeSimilarity matrix (via TreeDist::NyeSimilarity, normalize = TRUE) reproduces expected method separation
- On dropout-containing 1024-cell datasets, LinRace-TST/IST outperform all baselines on all three metrics (RF distance, Nye similarity, CID), matching Fig. 2 Source Data values
- Simulation parameters fall within the tested grid: mu in {0.025, 0.05, 0.075}, ncells in {1024, 2048, 4096}, 16 barcode characters, Sigma = 0.5, 500 genes at ge_prob = 0.3
- Aggregated benchmark CSV rows align per method and condition with the reference Fig. 2 Source Data table

## Limitations

- LinRace assumes irreversible cell state transitions; reversible transitions observed in real systems are not modeled
- Assumes all cell states during development exist in current-day cells; fast-changing systems may have ancestral states uncaptured by the scRNA-seq data
- Most applicable to systems with asynchronous differentiation; results depend on future improvements in lineage barcode technology with high diversity and coverage
- Software versions for the R packages are not stated in the methods, complicating exact reproduction

## Evidence

- [methods] Simulation grid: "true_counts_res <- CIF2Truecounts(ngenes = 500,ncif = n_cif,ge_prob = 0.3,ncells = ncells, cif_res = cif_res)"
- [methods] Baseline NJ: "dist_h <- dist.hamming(sim_data); Treenj = nj(dist_h)"
- [intro] Motivation: "However, inferring the cell division tree of a massive number of cells is a challenging problem."
- [readme] Install: [evidence span withheld — anchor status: external_doc]
- [discussion] Assumption caveat: "LinRace assumes that cell state transitions are irreversible, which is generally true under homeostatic conditions."
