---
name: species-mixing-metric-evaluation
description: Use when you have integrated single-cell data from two or more species
  in a joint latent space (e.g., via scVI, scANVI or SeuratV4) and need to quantify
  species mixing versus biology conservation to compare integration strategies.
when_to_use_negative:
- When cell type annotations are unreliable or literature cell type homology is unknown
  — the quantitative scoring system depends on these two assumptions
- When the goal is a non-one-to-one cell type mapping across species — the benchmark
  scoring uses one-to-one cell type mappings and valid non-one-to-one mappings were
  not assessed
- When evaluating whole-body atlas integration with challenging gene homology annotation,
  where SAMap-type approaches are more appropriate than flat homology-mapping scores
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scANVI
  role: Integration algorithm whose embeddings are scored for species mixing and biology
    conservation
- name: scVI
  role: Integration algorithm whose embeddings are scored for species mixing and biology
    conservation
- name: SeuratV4
  role: Integration algorithm whose embeddings are scored for species mixing and biology
    conservation
  repo: https://github.com/satijalab/seurat-wrappers
- name: BENGAL
  role: Benchmarking pipeline that runs integration and computes the 9 metrics plus
    the biology conservation metric
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:f50cdd14bfd7db4b81b29ef6c5e27f1c4ff1c59449ad033413adf3c4e4fb656b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat-wrappers
  tools:
  - BENGAL
  - SeuratV4
  - scANVI
  - scVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# species-mixing-metric-evaluation

## Summary

Evaluates cross-species single-cell RNA-seq integration quality by scoring homology-mapping × algorithm combinations with 9 established integration metrics plus a biology conservation metric for maintenance of cell type distinguishability, as implemented in the BENGAL pipeline.

## When to use

Use when you have integrated single-cell data from two or more species in a joint latent space (e.g., via scVI, scANVI or SeuratV4) and need to quantify species mixing versus biology conservation to compare integration strategies.

## When NOT to use

- When cell type annotations are unreliable or literature cell type homology is unknown — the quantitative scoring system depends on these two assumptions
- When the goal is a non-one-to-one cell type mapping across species — the benchmark scoring uses one-to-one cell type mappings and valid non-one-to-one mappings were not assessed
- When evaluating whole-body atlas integration with challenging gene homology annotation, where SAMap-type approaches are more appropriate than flat homology-mapping scores

## Inputs

- Jointly embedded multi-species single-cell RNA-seq data (latent space from scVI, scANVI or SeuratV4)
- Cell type annotations unified across species (e.g., via scOntoMatch ontology matching)
- Homologous-gene feature sets per homology strategy (one-to-one orthologs vs in-paralog-including HE/SH)
- Known cell type homology relationships from the literature used as ground truth

## Outputs

- Integrated and biology-conservation scores for each homology-method × algorithm combination across 9 established metrics plus the new biology conservation metric
- Aggregated per-homology-strategy comparison table (e.g., homology_strategy_comparison.csv)
- Ranking of integration strategies by species-mixing vs biology-conservation balance

## How to apply

Run the candidate integration algorithms (scVI, scANVI, SeuratV4) over each homologous-gene feature space (e.g., one-to-one orthologs versus in-paralog-including HE/SH strategies) to embed species in a joint latent space. Score every homology-method × algorithm combination with the 9 established integration metrics plus the paper's new biology conservation metric that quantifies maintenance of cell type distinguishability across species. Aggregate integrated and biology-conservation scores by homology method into a comparison table (e.g., homology_strategy_comparison.csv) and record which HVG sets each strategy selects. Judge balance by the benchmark's expectation that good methods achieve a balance between species-mixing and biology conservation; note that for evolutionarily distant pairs (e.g., Xenopus tropicalis vs Danio rerio), in-paralog-including strategies outperform one-to-one orthologs on biology conservation. Interpret scores under the stated assumptions that cell type annotations are accurate and known cell type homology is reliable.

## Related tools

- **scANVI** (Integration algorithm whose embeddings are scored for species mixing and biology conservation)
- **scVI** (Integration algorithm whose embeddings are scored for species mixing and biology conservation)
- **SeuratV4** (Integration algorithm whose embeddings are scored for species mixing and biology conservation) — https://github.com/satijalab/seurat-wrappers
- **BENGAL** (Benchmarking pipeline that runs integration and computes the 9 metrics plus the biology conservation metric)

## Evaluation signals

- The comparison table reproduces the benchmark's direction of results: for the evolutionarily distant Embryo_dr_xt task, in-paralog-including homology strategies (HE/SH) outperform one-to-one orthologs on biology conservation
- Scores are computed for all homology-method × algorithm combinations with the full set of 9 established metrics plus the biology conservation metric
- Top-scoring methods (scANVI, scVI, SeuratV4) show a balance between species-mixing and biology conservation rather than maximizing one metric at the expense of the other
- Each score row is tied to the HVG set selected under its homology strategy, enabling traceability of feature-space effects
- Biology conservation score maintains distinguishability of matched cell type homologs (cell ontology annotations matched cross-datasets, e.g., via scOntoMatch ontoMultiMatch)

## Limitations

- The quantitative scoring system assumes (1) cell type annotations are accurate and (2) known cell type homology from literature is correct
- Benchmark uses one-to-one cell type mappings; valid non-one-to-one mappings across species were not assessed due to lack of ground-truth datasets, sophisticated methods, and specialised metrics
- Cross-species integration relies on curated sequence-based gene orthology, ignoring species-specific genes; out-paralogs can show higher expression similarity than their orthologs through gain, loss and partition of function
- Expression similarity may arise from convergence or concerted evolution rather than homology; no algorithm in the benchmark disentangles these sources

## Evidence

- [abstract] Performance was assessed with 9 established metrics and a new biology conservation metric.: "using 9 established metrics. We also develop a new biology conservation metric to address the maintenance of cell type distinguishability."
- [abstract] Balanced methods identified by the benchmark.: "Overall, scANVI, scVI and SeuratV4 methods achieve a balance between species-mixing and biology conservation."
- [discussion] Assumptions underlying the scoring system.: "Our quantitative scoring system of species mixing and biology conservation is supported by two assumptions: (1) cell type annotations are accurate;"
- [abstract] In-paralogs benefit for distant species.: "For evolutionarily distant species, including in-paralogs is beneficial."
