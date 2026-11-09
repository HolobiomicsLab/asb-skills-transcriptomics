---
name: multi-strategy-performance-comparison
description: Use when several integration algorithms (SAMap, fastMNN, Harmony, LIGER,
  LIGER UINMF, Scanorama, SeuratV4, scVI, scANVI) must be compared on identical cross-species
  scRNA-seq inputs with a common scoring scheme.
when_to_use_negative:
- When the goal is a single best integration of one species pair rather than a benchmark
  — a comparator tuned to the setting (e.g. scANVI, scVI or SeuratV4 for balanced
  mixing/conservation) may suffice
- When ground-truth cell type annotations or confident known cell type homology are
  unavailable — the paper states quantitative scoring assumes accurate annotations
  and known homology
- When non-one-to-one cell type mappings across species must be assessed — the benchmark
  is limited to one-to-one cell type mappings
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SAMap
  role: Integration strategy benchmarked on whole-body atlases with challenging gene
    homology annotation; builds a de-novo reciprocal BLAST gene-gene homology graph
    bypassing curated orthology
  repo: https://github.com/atarashansky/SAMap
- name: fastMNN
  role: Mutual-nearest-neighbours comparator integration strategy run with curated
    homologous-gene mappings
  repo: https://github.com/LTLA/batchelor
- name: Harmony
  role: Iterative-clustering comparator integration strategy
- name: LIGER
  role: iNMF-based comparator integration strategy
  repo: https://github.com/welch-lab/liger
- name: LIGER UINMF
  role: UINMF comparator that also takes unshared features
  repo: https://github.com/welch-lab/liger
- name: Scanorama
  role: Panorama-stitching comparator integration strategy
- name: SeuratV4
  role: Comparator strategy reported to balance species-mixing and biology conservation
  repo: https://github.com/satijalab/seurat-wrappers
- name: scVI
  role: Comparator strategy reported to balance species-mixing and biology conservation
- name: scANVI
  role: Comparator strategy reported to balance species-mixing and biology conservation
- name: BENGAL
  role: Benchmarking pipeline that loads data, runs strategies, and computes species-mixing
    and biology-conservation metrics
  repo: https://github.com/Papatheodorou-Group/scOntoMatch
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:497b9d633b573e6370032a56db75a5c30460fda30614f3be2364356b2216609c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/atarashansky/SAMap
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
  - scVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# multi-strategy-performance-comparison

## Summary

Benchmark multiple cross-species single-cell RNA-seq integration strategies head-to-head under the BENGAL pipeline, scoring each with the same metrics (e.g. alignment score as the average percentage of cross-species kNN neighbours, plus biology-conservation metrics such as trajectory conservation) on a shared task like the Embryo_xt_dr xenopus–zebrafish whole-body atlas.

## When to use

Use when several integration algorithms (SAMap, fastMNN, Harmony, LIGER, LIGER UINMF, Scanorama, SeuratV4, scVI, scANVI) must be compared on identical cross-species scRNA-seq inputs with a common scoring scheme.

## When NOT to use

- When the goal is a single best integration of one species pair rather than a benchmark — a comparator tuned to the setting (e.g. scANVI, scVI or SeuratV4 for balanced mixing/conservation) may suffice
- When ground-truth cell type annotations or confident known cell type homology are unavailable — the paper states quantitative scoring assumes accurate annotations and known homology
- When non-one-to-one cell type mappings across species must be assessed — the benchmark is limited to one-to-one cell type mappings

## Inputs

- Raw count matrices for each species in the benchmark task (e.g. xenopus and zebrafish whole-body embryo atlas), loaded into per-species AnnData or Seurat objects
- Curated homologous-gene mappings for comparator strategies
- Per-cell metadata: species label, cell type / ontology annotation, and developmental stage labels for trajectory-conservation scoring

## Outputs

- Tabulated alignment score (average % cross-species kNN neighbours) per integration strategy
- Trajectory conservation score per strategy on the embryo task
- Comparison figures/tables of SAMap versus comparator strategies (e.g. Supplementary Fig. 17 pattern)

## How to apply

Instantiate a common benchmark task (e.g. the Embryo_xt_dr two-species whole-body atlas between xenopus and zebrafish) and prepare per-species AnnData objects from the raw count matrices. Run SAMap building a de-novo reciprocal BLAST gene-gene homology graph (NCBI BLAST on PATH; BLAST can take up to ~4 hours), while running each comparator (fastMNN, Harmony, LIGER, LIGER UINMF, Scanorama, SeuratV4, scVI, scANVI) with curated homologous-gene mappings. Compute an identical alignment score for every strategy — the average percentage of cross-species neighbours among each cell's kNN in the integrated space — so that species-mixing is comparable. Add biology-conservation metrics such as the trajectory conservation score for stage progression to detect over-mixing. Tabulate alignment and conservation scores per strategy and render per-task comparisons (e.g. Supplementary Fig. 17); here SAMap should show a significantly higher alignment score than the comparators while retaining a low trajectory conservation score (Supplementary Fig. 19), matching the finding that scANVI, scVI and SeuratV4 instead balance species-mixing and biology conservation.

## Related tools

- **SAMap** (Integration strategy benchmarked on whole-body atlases with challenging gene homology annotation; builds a de-novo reciprocal BLAST gene-gene homology graph bypassing curated orthology) — https://github.com/atarashansky/SAMap
- **fastMNN** (Mutual-nearest-neighbours comparator integration strategy run with curated homologous-gene mappings) — https://github.com/LTLA/batchelor
- **Harmony** (Iterative-clustering comparator integration strategy)
- **LIGER** (iNMF-based comparator integration strategy) — https://github.com/welch-lab/liger
- **LIGER UINMF** (UINMF comparator that also takes unshared features) — https://github.com/welch-lab/liger
- **Scanorama** (Panorama-stitching comparator integration strategy)
- **SeuratV4** (Comparator strategy reported to balance species-mixing and biology conservation) — https://github.com/satijalab/seurat-wrappers
- **scVI** (Comparator strategy reported to balance species-mixing and biology conservation)
- **scANVI** (Comparator strategy reported to balance species-mixing and biology conservation)
- **BENGAL** (Benchmarking pipeline that loads data, runs strategies, and computes species-mixing and biology-conservation metrics) — https://github.com/Papatheodorou-Group/scOntoMatch

## Examples

```
from samap import SAMAP; sm = SAMAP(sams={'sp1': 'xenopus.h5ad', 'sp2': 'zebrafish.h5ad'}, f_maps='maps/'); sm.run()
```

## Evaluation signals

- SAMap achieves a significantly higher alignment score (average % cross-species kNN neighbours) than fastMNN, Harmony, LIGER, LIGER UINMF, Scanorama, SeuratV4, scVI and scANVI on the whole-body embryo task, matching Supplementary Fig. 17
- SAMap retains a low trajectory conservation score on the xenopus–zebrafish embryo task (Supplementary Fig. 19), demonstrating that high alignment does not imply stage-progression conservation
- Every strategy is scored with the identical metric definition on the same integrated space, so per-strategy scores are directly comparable in the tabulated output
- Comparator strategies use curated homologous-gene mappings while SAMap uses its de-novo reciprocal BLAST graph — confirm the homology source matches the assigned strategy in the results table
- scANVI, scVI and SeuratV4 show balanced species-mixing and biology conservation relative to other strategies, as reported in the abstract

## Limitations

- Quantitative scoring assumes accurate cell type annotations and confident known cell type homology from literature
- Benchmark is limited to one-to-one cell type mappings; valid non-one-to-one mappings across species are not assessed
- Cross-species integration relies on curated sequence-based gene orthology, ignoring species-specific genes
- SAMap is heuristic and does not explain observed cell type expression similarity
- Expression similarities across species may arise from convergence or concerted evolution rather than homology; no algorithm disentangles these sources

## Evidence

- [abstract] SAMap outperforms when integrating whole-body atlases between species with challenging gene homology annotation.: "SAMap outperforms when integrating whole-body atlases between species with challenging gene homology annotation."
- [abstract] scANVI scVI SeuratV4 balance: "Overall, scANVI, scVI and SeuratV4 methods achieve a balance between species-mixing and biology conservation."
- [supplementary] workflow alignment score: [evidence span withheld — anchor status: not_found]
- [supplementary] workflow homology graph: [evidence span withheld — anchor status: not_found]
- [discussion] discussion scoring assumptions: "Our quantitative scoring system of species mixing and biology conservation is supported by two assumptions: (1) cell type annotations are accurate; (2) known cell type homology from literature is"
- [readme] README BLAST runtime: [evidence span withheld — anchor status: external_doc]
- [readme] README SAMap invocation: [evidence span withheld — anchor status: external_doc]
