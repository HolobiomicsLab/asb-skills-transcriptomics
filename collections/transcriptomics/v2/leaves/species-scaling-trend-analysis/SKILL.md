---
name: species-scaling-trend-analysis
description: Use when evaluating how the number of species involved in a cross-species
  integration affects cell type distinguishability, e.g. detecting overcorrection
  of few-species cell types by nearest-neighbour methods via iso_F1 trends.
when_to_use_negative:
- When the goal is to measure species mixing rather than biology conservation — iso_F1
  addresses maintenance of cell type distinguishability, not how well species are
  mixed in the embedding
- When cell type annotations or known cell type homology are unreliable, since the
  metric depends on accurate annotations and confident one-to-one homology
- When the mapping across species is non-one-to-one or cell type correspondence needs
  hierarchical/complex description, which this flat-mapping-based metric does not
  cover
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BENGAL
  role: Pipeline that generates the per-strategy integrated embeddings and provides
    the assessment framework in which iso_F1 is computed per species subset
- name: fastMNN
  role: Nearest-neighbour-based integration algorithm whose iso_F1 trend is evaluated
    for overcorrection of few-species cell types
  repo: https://github.com/LTLA/batchelor
- name: SeuratV4
  role: CCA and RPCA integration strategies benchmarked for their iso_F1 trend as
    species are added
  repo: https://github.com/satijalab/seurat-wrappers
- name: LIGER
  role: iNMF-based integration strategy evaluated in the species-scaling comparison
  repo: https://github.com/welch-lab/liger
- name: LIGER UINMF
  role: UINMF (unshared features) variant evaluated in the species-scaling comparison,
    including cross-species analyses
  repo: https://github.com/welch-lab/liger
- name: SAMap
  role: Integration strategy included in the benchmark, outperforming for whole-body
    atlases with challenging gene homology annotation
  repo: https://github.com/atarashansky/SAMap
- name: scOntoMatch
  role: Ontology matching tool used to align cell type homology mappings that define
    the shared versus isolated cell type labels scored by iso_F1
  repo: https://github.com/Papatheodorou-Group/scOntoMatch
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:b8418e088bf7ccdd2bf31b0df1d793e99a0130f431f1fd031bfb3a1d232bdef3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LTLA/batchelor
  tools:
  - BENGAL
  - Harmony
  - LIGER
  - LIGER UINMF
  - SAMap
  - Scanorama
  - SeuratV4
  - fastMNN
  - scANVI
  - scOntoMatch
  - scVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# species-scaling-trend-analysis

## Summary

Computes and plots how biology-conservation metrics, especially isolated-label F1 (iso_F1), change as more species are added to a cross-species scRNA-seq integration, using the BENGAL pipeline's 28 gene-homology-mapping x integration-algorithm strategies.

## When to use

Use when evaluating how the number of species involved in a cross-species integration affects cell type distinguishability, e.g. detecting overcorrection of few-species cell types by nearest-neighbour methods via iso_F1 trends.

## When NOT to use

- When the goal is to measure species mixing rather than biology conservation — iso_F1 addresses maintenance of cell type distinguishability, not how well species are mixed in the embedding
- When cell type annotations or known cell type homology are unreliable, since the metric depends on accurate annotations and confident one-to-one homology
- When the mapping across species is non-one-to-one or cell type correspondence needs hierarchical/complex description, which this flat-mapping-based metric does not cover

## Inputs

- Per-strategy integrated embeddings for each cumulative species subset (28 gene-homology-mapping x integration-algorithm combinations from BENGAL)
- Four-species heart scRNA-seq datasets with shared cell type homology mapping (22 cell types, 8 shared by all species)
- Cell type annotations aligned across species via the cell type homology mapping

## Outputs

- iso_F1 table per strategy per number-of-species condition (iso_f1_heart.csv)
- iso_F1 versus number-of-species trend plot per strategy (fig6c_iso_f1_trend.png)
- Identification of strategies that overcorrect few-species cell types (e.g. fastMNN, SeuratV4 CCA/RPCA)

## How to apply

Load the per-strategy integrated embeddings produced with the BENGAL pipeline (28 combinations of gene homology mapping and integration algorithms such as fastMNN, SeuratV4 CCA/RPCA, scVI, scANVI, Harmony, LIGER, LIGER UINMF, Scanorama, SAMap) for the cumulative species subsets. For each strategy and each number-of-species condition, compute the isolated-label F1 (iso_F1) — the biology conservation metric for maintenance of cell type distinguishability — against the shared cell type homology mapping (heart: only 8 of 22 cell types are shared by all species). Tabulate iso_F1 per strategy per species count (e.g. write iso_f1_heart.csv) and plot iso_F1 versus number of species per strategy (fig6c_iso_f1_trend.png). Decision point: a downward iso_F1 trend with increasing species count signals overcorrection, where nearest-neighbour-based methods (fastMNN, SeuratV4 CCA, SeuratV4 RPCA) merge similar cell types found in only a few species datasets, whereas balanced strategies like scANVI, scVI and SeuratV4 retain distinguishability.

## Related tools

- **BENGAL** (Pipeline that generates the per-strategy integrated embeddings and provides the assessment framework in which iso_F1 is computed per species subset)
- **fastMNN** (Nearest-neighbour-based integration algorithm whose iso_F1 trend is evaluated for overcorrection of few-species cell types) — https://github.com/LTLA/batchelor
- **SeuratV4** (CCA and RPCA integration strategies benchmarked for their iso_F1 trend as species are added) — https://github.com/satijalab/seurat-wrappers
- **LIGER** (iNMF-based integration strategy evaluated in the species-scaling comparison) — https://github.com/welch-lab/liger
- **LIGER UINMF** (UINMF (unshared features) variant evaluated in the species-scaling comparison, including cross-species analyses) — https://github.com/welch-lab/liger
- **SAMap** (Integration strategy included in the benchmark, outperforming for whole-body atlases with challenging gene homology annotation) — https://github.com/atarashansky/SAMap
- **scOntoMatch** (Ontology matching tool used to align cell type homology mappings that define the shared versus isolated cell type labels scored by iso_F1) — https://github.com/Papatheodorou-Group/scOntoMatch

## Examples

```
R: iso_f1_heart <- compute_iso_F1(embeddings, mapping, n_species_subsets); write.csv(iso_f1_heart, 'iso_f1_heart.csv'); ggsave('fig6c_iso_f1_trend.png', plot_iso_f1_trend(iso_f1_heart))
```

## Evaluation signals

- Reproduced iso_F1 trend qualitatively matches the published Fig. 6c pattern: nearest-neighbour-based methods (fastMNN, SeuratV4 CCA/RPCA) lose cell type distinguishability as species count increases
- Per-strategy iso_F1 scores match the reported ranking across strategies
- iso_F1 is computed for every strategy and every cumulative species subset against the shared cell type homology mapping (heart: 8 of 22 cell types shared by all species)
- Output table contains one iso_F1 value per strategy per number-of-species condition and serialises to iso_f1_heart.csv without missing conditions
- Low iso_F1 for SeuratV4 CCA confirms that few-species cell types are not well-separated, consistent with the reported overcorrection tendency

## Limitations

- Quantitative scoring assumes accurate cell type annotations and confident known cell type homology from literature
- Benchmark is limited to one-to-one cell type mappings; valid non-one-to-one mappings across species are not assessed
- Cross-species integration relies on curated sequence-based gene orthology, ignoring species-specific genes
- Expression similarities across species may arise from convergence or concerted evolution rather than homology; flat mappings do not comprehensively describe cross-species cell type correspondence

## Evidence

- [intro] Research question: how the number of species involved and species divergence time impact integration output differently among strategies.: "the number of species involved in the integration and the time that species have diverged can impact the output characteristics differently among strategies"
- [abstract] Biology conservation metric for cell type distinguishability.: "using 9 established metrics. We also develop a new biology conservation metric to address the maintenance of cell type distinguishability."
- [abstract] Balanced strategies retain biology conservation.: "Overall, scANVI, scVI and SeuratV4 methods achieve a balance between species-mixing and biology conservation."
- [intro] BENGAL pipeline definition.: "we develop the BENchmarking strateGies for cross-species integrAtion of singLe-cell RNA sequencing data (BENGAL) pipeline"
- [readme] LIGER supports cross-species integration contexts.: [evidence span withheld — anchor status: external_doc]
