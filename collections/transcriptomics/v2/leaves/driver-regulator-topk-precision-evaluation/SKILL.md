---
name: driver-regulator-topk-precision-evaluation
description: Use when benchmarking a ranked list of predicted driver regulators against
  ground-truth gene sets (e.g., GO:0045165, GO:0019827, GO:0007492) by computing top-k
  precision for k=1–20 and comparing multiple methods' precision curves.
when_to_use_negative:
- The goal is to evaluate GRN edge inference quality rather than driver regulator
  identification — benchmarking tests distinguish these tasks, and this skill only
  covers driver regulator precision
- No ground-truth gene sets are available for the cell type or lineage of interest
  (the article notes label datasets serving as ground-truths are often lacking)
- The input is raw scRNA-seq expression data rather than an already-computed driver
  regulator ranking — use the CEFCON GRN inference and driver_regulators() steps first
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CEFCON
  role: Source of the driver regulator ranking lists (with influence scores) being
    benchmarked against baselines
  repo: https://github.com/WPZgithub/CEFCON
- name: Python
  role: Implementation language for computing top-k precision by intersecting ranked
    top-k lists with ground-truth gene sets
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
  source_content_hash: sha256:12d049562d3173ed25fad15c7606ff272a0512e9a8bbc1cfbeca3b6377699c0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - CEFCON
  - Python
  - matplotlib
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# driver-regulator-topk-precision-evaluation

## Summary

Evaluates driver regulator identification methods (CEFCON vs. VIPER, ANANSE, SCENIC, CellRouter, CellOracle) by computing top-k precision (k=1–20) of their ranked driver lists against GO-term and curated ground-truth gene sets on mESC and hESC datasets.

## When to use

Use when benchmarking a ranked list of predicted driver regulators against ground-truth gene sets (e.g., GO:0045165, GO:0019827, GO:0007492) by computing top-k precision for k=1–20 and comparing multiple methods' precision curves.

## When NOT to use

- The goal is to evaluate GRN edge inference quality rather than driver regulator identification — benchmarking tests distinguish these tasks, and this skill only covers driver regulator precision
- No ground-truth gene sets are available for the cell type or lineage of interest (the article notes label datasets serving as ground-truths are often lacking)
- The input is raw scRNA-seq expression data rather than an already-computed driver regulator ranking — use the CEFCON GRN inference and driver_regulators() steps first

## Inputs

- Driver regulator ranking lists with influence scores for CEFCON and baselines (VIPER, ANANSE, SCENIC, CellRouter, CellOracle) from Zenodo record 7564872 (mESC and hESC datasets)
- Ground-truth positive gene sets from GO terms GO:0045165 (cell fate commitment), GO:0019827 (stem cell population maintenance), GO:0007492 (endoderm development)
- Literature-curated driver regulator gene sets

## Outputs

- Top-k precision table (k=1..20) for all six methods against all four ground-truth gene sets
- Top-k precision curve plots (analogous to Fig. 4a-h) rendered with matplotlib
- Verified rank positions of key drivers (e.g., NANOG, SOX2, POU5F1) in the CEFCON hESC ranking

## How to apply

Download the driver regulator ranking lists with influence scores for CEFCON and the five baselines (VIPER, ANANSE, SCENIC, CellRouter, CellOracle) from the Zenodo record 7564872 for the hESC (or mESC) dataset. Assemble ground-truth positive gene sets from GO terms GO:0045165 (cell fate commitment), GO:0019827 (stem cell population maintenance), GO:0007492 (endoderm development), plus literature-curated driver genes. In Python, for each method and each ground-truth set, intersect the ranked top-k driver list (k=1..20) with the positive set and divide by k to obtain top-k precision. Aggregate the precision values into a single table across all six methods and four gene sets, then plot precision curves with matplotlib (analogous to Fig. 4a-h). Correctness hinges on CEFCON's hESC ranking reproducing NANOG at rank 1, SOX2 at rank 2, and POU5F1 at rank 5.

## Related tools

- **CEFCON** (Source of the driver regulator ranking lists (with influence scores) being benchmarked against baselines) — https://github.com/WPZgithub/CEFCON
- **Python** (Implementation language for computing top-k precision by intersecting ranked top-k lists with ground-truth gene sets)
- **matplotlib** (Optional dependency for plotting top-k precision curves analogous to Fig. 4a-h)

## Examples

```
precision_k = [len(set(ranked_drivers[:k]) & ground_truth_set) / k for k in range(1, 21)]
```

## Evaluation signals

- The reproduced CEFCON hESC ranking places NANOG at rank 1, SOX2 at rank 2, and POU5F1 at rank 5 within the top-20 driver regulators
- The top-k precision table contains values for all six methods (CEFCON, VIPER, ANANSE, SCENIC, CellRouter, CellOracle) across all four ground-truth gene sets
- CEFCON's top-k precision curves are the highest among all six methods for k=1..20, with the highest-ranked genes almost perfectly identified for all four ground-truth sets
- Precision values fall in [0,1] and are non-increasing-or-noisy over increasing k but capped by k / min(k, |positive set|)
- Ground-truth sets cover exactly the three GO terms plus curated genes: GO:0045165, GO:0019827, GO:0007492

## Limitations

- Ground-truth label datasets are often lacking, especially for specific cell types or lineages, limiting where this evaluation can be applied
- Evaluation is dataset-specific: reported ranks (NANOG 1/20, SOX2 2/20, POU5F1 5/20) are validated only on the hESC dataset, and results depend on the Zenodo-recorded ranking lists
- CEFCON's underlying driver regulator identification can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously

## Evidence

- [abstract] Benchmarking tests demonstrated CEFCON's superiority over baseline methods in GRN construction, driver regulator identification, and gene module identification: "Extensive benchmarking tests consistently demonstrated the superiority of CEFCON in GRN construction, driver regulator identification, and gene module identification over baseline methods"
- [methods] Zenodo data availability: "All the input data mentioned in the paper can be downloaded from [here](https://zenodo.org/record/7564872)"
- [methods] Ground-truth GO terms used in the task card: [evidence span withheld — anchor status: not_found]
- [readme] matplotlib as optional visualization dependency: "### Optional (for performance evaluation, visualization and other analyses)
- matplotlib(>=3.5.3)"
- [discussion] Gap in ground-truth availability: "the label datasets serving as ground-truths are often lacking, especially for specific cell types or lineages"
