---
name: random-seed-replicate-aggregation
description: Use when the method being benchmarked involves stochastic steps (random
  clustering, random subsampling) and you need seed-averaged scores or robustness
  estimates across parameter sweeps.
when_to_use_negative:
- The pipeline is fully deterministic (no random clustering or sampling steps), so
  seed replicates would be identical and add no information.
- Trajectories are disconnected or contain cycles, since Totem cannot generate them
  and the sweep's scoring step would be invalid.
- The dataset has extremely rare plus extremely prevalent cell types simultaneously,
  where k-medoids fails regardless of seed and aggregation would average over structurally
  broken clusterings.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Totem
  role: Stochastic CLARA k-medoids clustering and cell-connectivity-guided trajectory
    inference being replicated across seeds
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Constructs the Minimum Spanning Tree trajectory backbone from the clustering
    selected in each seeded run
- name: SingleCellExperiment
  role: Container format for the benchmark datasets loaded before PrepareTotem preprocessing
- name: dyndimred
  role: Provides LMDS dimensionality reduction used in the pipeline being replicated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:49705422ad4106b01805089e316755711dea85928187a111b39e2a89ab2222df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - R
  - SingleCellExperiment
  - Slingshot
  - Totem
  - dyndimred
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# random-seed-replicate-aggregation

## Summary

Run a stochastic analysis (e.g., Totem's CLARA k-medoids clustering) repeatedly with different random seeds and aggregate the results across replicates to separate method robustness from seed-level noise.

## When to use

Use when the method being benchmarked involves stochastic steps (random clustering, random subsampling) and you need seed-averaged scores or robustness estimates across parameter sweeps.

## When NOT to use

- The pipeline is fully deterministic (no random clustering or sampling steps), so seed replicates would be identical and add no information.
- Trajectories are disconnected or contain cycles, since Totem cannot generate them and the sweep's scoring step would be invalid.
- The dataset has extremely rare plus extremely prevalent cell types simultaneously, where k-medoids fails regardless of seed and aggregation would average over structurally broken clusterings.

## Inputs

- 10 single-cell benchmark datasets from the dynverse Zenodo deposit (doi:10.5281/zenodo.1443566) as SingleCellExperiment objects
- Parameter grid: number of CLARA k-medoids clusterings (100, 500, 1000, 2500, 5000, 7500) and upper limit of the k range (5, 10, 15, 20, 30, 40)
- 10 random seeds per configuration

## Outputs

- Per-run dynverse overall scores (one entry per dataset, clustering-count value, k-range upper limit, and seed)
- Aggregated (seed-averaged) overall scores per configuration and dataset, e.g., benchmark_overall_scores.csv reproducing Supplementary Figures S2/S3
- Robustness assessment of performance vs. number of clusterings and k upper limit

## How to apply

For each dataset and each parameter configuration (e.g., Totem's number of CLARA clusterings in {100, 500, 1000, 2500, 5000, 7500} and upper k limit in {5, 10, 15, 20, 30, 40}), run the full pipeline — PrepareTotem preprocessing, clustering with the min.cluster.size filter (default 5 cells per cluster), Slingshot MST construction, and dynverse metric scoring — 10 times with 10 different random seeds. Then aggregate the dynverse overall scores across the 10 seeds for each (dataset, configuration) pair. This mirrors the Totem robustness sweep (Supplementary Figures S2/S3), where variation in overall scores shrank as the number of clusterings increased and stabilized above 2500 clusterings (5000 or 7500). Tabulate one overall-score entry per dataset, configuration value, and seed so aggregation is fully traceable.

## Related tools

- **Totem** (Stochastic CLARA k-medoids clustering and cell-connectivity-guided trajectory inference being replicated across seeds) — https://github.com/elolab/Totem
- **Slingshot** (Constructs the Minimum Spanning Tree trajectory backbone from the clustering selected in each seeded run)
- **SingleCellExperiment** (Container format for the benchmark datasets loaded before PrepareTotem preprocessing)
- **dyndimred** (Provides LMDS dimensionality reduction used in the pipeline being replicated)

## Examples

```
Rscript totem_robustness_sweep.R --datasets 10 --n_clusterings 100,500,1000,2500,5000,7500 --k_max 5,10,15,20,30,40 --seeds 10 --min.cluster.size 5 --out benchmark_overall_scores.csv
```

## Evaluation signals

- The output table contains exactly one overall-score entry per (dataset, clustering-count value, k-range upper limit, seed) combination
- All 10 benchmark datasets from Zenodo deposit 10.5281/zenodo.1443566 are covered
- All recorded dynverse overall scores fall within the valid [0, 1] range
- Aggregated scores show small variation across seeds when the number of clusterings exceeds 2500 (5000 or 7500), consistent with the paper's robustness finding
- Each seeded run applied the min.cluster.size filter (default 5 cells per cluster) to discard clusterings with too-small clusters

## Limitations

- Totem's CLARA clustering is stochastic, so single-seed results are not reproducible without seed replication; variation across seeds is largest at low numbers of clusterings (e.g., 100).
- k-medoids may be unstable with high-dimensional PCA input (e.g., 30 principal components), producing 1-2 cell clusters and corrupting scores for affected seeds.
- No correct pseudotime support for multi-root (converging) trajectories via dynplot, which limits scoring of some seeded runs.
- Aggregated scores reflect Totem only; seed-replicate behavior of other trajectory inference methods may differ.

## Evidence

- [supplementary] Totem's robustness improved as the number of clusterings increased, with small variation when the number of clusterings exceeded 2500 (5000 or 7500).: [evidence span withheld — anchor status: not_found]
- [supplementary] applying the min.cluster.size filter (default 5 cells per cluster) to discard clusterings with too-small clusters: [evidence span withheld — anchor status: not_found]
- [intro] Totem generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering.: "`Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
- [intro] the next step is to perform clustering that involves running the k-medoids (CLARA) algorithm a large number of times (default 10k).: "the next step is to perform clustering that involves running the k-medoids (CLARA) algorithm a large number of times (default 10k)."
- [discussion] Single-cell -omics datasets containing a trajectory (2.0.0) [Data set]. Zenodo. [https://doi.org/10.5281/zenodo.1443566]: "Single-cell -omics datasets containing a trajectory (2.0.0) [Data set]. Zenodo. [https://doi.org/10.5281/zenodo.1443566]"
