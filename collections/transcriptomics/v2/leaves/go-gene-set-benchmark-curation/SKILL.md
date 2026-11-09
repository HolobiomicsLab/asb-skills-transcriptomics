---
name: go-gene-set-benchmark-curation
description: Use when a ranked list of predicted regulators (e.g., CEFCON vs VIPER/ANANSE/SCENIC/CellRouter/CellOracle)
  must be validated against GO biological-process gene sets such as GO:0045165, GO:0019827,
  and GO:0007492.
when_to_use_negative:
- The goal is GRN edge inference accuracy rather than driver regulator identification
  — this skill evaluates only ranked gene-level driver predictions.
- No curated or GO ground-truth gene sets exist for the specific cell type or lineage
  under study, as the article notes ground-truth labels are often lacking for specific
  cell types or lineages.
- The candidate method outputs unranked gene sets without scores, since top-k precision
  requires an ordered ranking.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3308
tools:
- name: CEFCON
  role: Source of the driver regulator ranking (via influence scores from network
    control theory) being benchmarked; Python package installed from its GitHub repo
  repo: https://github.com/WPZgithub/CEFCON
- name: Python
  role: Implementation language for intersecting top-k ranked driver lists with GO
    ground-truth sets and computing precision
  repo: https://github.com/WPZgithub/CEFCON
- name: matplotlib
  role: Optional dependency for plotting top-k precision curves analogous to Fig.
    4a-h
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:78cae38f944b4f3562418040feac9227d7c40d27b9ea464895650eab7e586744
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - CEFCON
  - Python
  - matplotlib
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# go-gene-set-benchmark-curation

## Summary

Assembles ground-truth positive gene sets from Gene Ontology terms and literature-curated driver genes to benchmark driver regulator identification methods, then scores ranked predictions by top-k precision (k=1–20).

## When to use

Use when a ranked list of predicted regulators (e.g., CEFCON vs VIPER/ANANSE/SCENIC/CellRouter/CellOracle) must be validated against GO biological-process gene sets such as GO:0045165, GO:0019827, and GO:0007492.

## When NOT to use

- The goal is GRN edge inference accuracy rather than driver regulator identification — this skill evaluates only ranked gene-level driver predictions.
- No curated or GO ground-truth gene sets exist for the specific cell type or lineage under study, as the article notes ground-truth labels are often lacking for specific cell types or lineages.
- The candidate method outputs unranked gene sets without scores, since top-k precision requires an ordered ranking.

## Inputs

- Driver regulator ranking lists with influence scores for each method (CEFCON, VIPER, ANANSE, SCENIC, CellRouter, CellOracle), from Zenodo record 7564872
- GO term gene sets: GO:0045165 (cell fate commitment), GO:0019827 (stem cell population maintenance), GO:0007492 (endoderm development)
- Literature-curated driver gene lists for the mESC and hESC datasets

## Outputs

- Top-k precision table (k=1..20) per method per ground-truth gene set
- Top-k precision curves plot (matplotlib, analogous to Fig. 4a-h)
- Verified ranking positions of canonical drivers (NANOG, SOX2, POU5F1) within the top-20

## How to apply

Build positive gene sets from GO terms relevant to the studied lineage (GO:0045165 cell fate commitment, GO:0019827 stem cell population maintenance, GO:0007492 endoderm development) plus literature-curated driver genes. Load per-method driver regulator ranking lists with influence scores (e.g., from the Zenodo record zenodo.org/record/7564872 for the CEFCON paper's hESC/mESC benchmarks). For each method and each ground-truth set, compute top-k precision for k=1..20 by intersecting the ranked top-k driver list with the positive set, then aggregate all precision curves into a single table for plotting (analogous to Fig. 4a-h with matplotlib). Rationale: the article notes ground-truth label datasets for supervised GRN alternatives are often lacking, so GO/curated gene sets provide the reference standard; the benchmark is accepted when the reproduced ranking recovers known drivers, e.g., NANOG at rank 1, SOX2 at rank 2, POU5F1 at rank 5 in the CEFCON hESC top-20.

## Related tools

- **CEFCON** (Source of the driver regulator ranking (via influence scores from network control theory) being benchmarked; Python package installed from its GitHub repo) — https://github.com/WPZgithub/CEFCON
- **Python** (Implementation language for intersecting top-k ranked driver lists with GO ground-truth sets and computing precision) — https://github.com/WPZgithub/CEFCON
- **matplotlib** (Optional dependency for plotting top-k precision curves analogous to Fig. 4a-h)

## Examples

```
prec = {m: [len(set(rank[m][:k]) & positives)/k for k in range(1,21)] for m in ['CEFCON','VIPER','ANANSE','SCENIC','CellRouter','CellOracle']}
```

## Evaluation signals

- The reproduced CEFCON hESC top-20 contains NANOG at rank 1, SOX2 at rank 2, and POU5F1 at rank 5.
- The top-k precision table contains values for all six methods (CEFCON, VIPER, ANANSE, SCENIC, CellRouter, CellOracle) and all four ground-truth gene sets (three GO terms plus curated genes).
- Precision values are bounded in [0,1] and are non-degenerate (nonzero for k=1..20 where the ground-truth set is nonempty).
- Precision curves show CEFCON above all five baselines across k=1..20, consistent with the article's reported superiority.

## Limitations

- Ground-truth label datasets for supervised evaluation are often lacking, especially for specific cell types or lineages.
- Benchmark coverage depends on GO annotation completeness and the scope of literature-curated driver genes for the chosen datasets (mESC, hESC).

## Evidence

- [abstract] superiority over baselines: "Extensive benchmarking tests consistently demonstrated the superiority of CEFCON in GRN construction, driver regulator identification, and gene module identification"
- [discussion] ground-truth scarcity: "the label datasets serving as ground-truths are often lacking, especially for specific cell types or lineages"
- [intro] control-based driver ranking: "CEFCON characterizes the cell fate dynamics based on the network control theory and identifies the driver regulators through combining the control-based methods with our proposed influence score"
- [readme] tool scope: "CEFCON is a computational tool for deciphering driver regulators of cell fate decisions from single-cell RNA-seq data."
