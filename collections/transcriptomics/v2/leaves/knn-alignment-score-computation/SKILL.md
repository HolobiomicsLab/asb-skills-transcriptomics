---
name: knn-alignment-score-computation
description: Use when quantifying how well a cross-species integration (e.g. Embryo_xt_dr
  whole-body atlas of xenopus and zebrafish) mixes cells across species in an integrated
  kNN graph or embedding.
when_to_use_negative:
- When the goal is to assess biology conservation (e.g. stage/cell-type trajectory
  maintenance) rather than species mixing — use a dedicated conservation metric, since
  SAMap's low trajectory conservation coexisted with a high alignment score
- When cell type annotations or known cell type homology are unreliable, since the
  benchmark's quantitative scoring assumes accurate annotations and confident cell
  type homology
- When only species-specific genes exist with no homology graph or integrated embedding
  — no cross-species neighbours can be computed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SAMap
  role: Produces the de-novo reciprocal BLAST homology-based whole-body integration
    whose kNN alignment score is computed; expected to achieve the highest alignment
    score
  repo: https://github.com/atarashansky/SAMap
- name: fastMNN
  role: Comparator integration strategy (mutual nearest neighbours) scored with the
    same alignment metric
  repo: https://github.com/LTLA/batchelor
- name: Harmony
  role: Comparator iterative-clustering integration strategy scored with the same
    alignment metric
- name: LIGER
  role: Comparator iNMF-based integration strategy scored with the same alignment
    metric
  repo: https://github.com/welch-lab/liger
- name: LIGER UINMF
  role: Comparator unshared-features iNMF integration strategy scored with the same
    alignment metric
  repo: https://github.com/welch-lab/liger
- name: Scanorama
  role: Comparator panorama-stitching integration strategy scored with the same alignment
    metric
- name: SeuratV4
  role: Comparator integration strategy scored with the same alignment metric; balances
    species-mixing and biology conservation
  repo: https://github.com/satijalab/seurat-wrappers
- name: scVI
  role: Comparator deep-generative integration strategy scored with the same alignment
    metric
- name: scANVI
  role: Comparator label-informed deep-generative integration strategy scored with
    the same alignment metric
- name: BENGAL
  role: Benchmarking pipeline in which the alignment score and companion biology conservation
    metric are implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:0b8429558fce1a5db6bfb614a047e0ef6a8076e5b0f8de9f39131c1ae8512359
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# knn-alignment-score-computation

## Summary

Computes the species-mixing (alignment) score of a cross-species single-cell integration as the average percentage of cross-species neighbours among each cell's k-nearest neighbours in the integrated embedding, as used in the BENGAL benchmark to compare SAMap against comparator integration strategies.

## When to use

Use when quantifying how well a cross-species integration (e.g. Embryo_xt_dr whole-body atlas of xenopus and zebrafish) mixes cells across species in an integrated kNN graph or embedding.

## When NOT to use

- When the goal is to assess biology conservation (e.g. stage/cell-type trajectory maintenance) rather than species mixing — use a dedicated conservation metric, since SAMap's low trajectory conservation coexisted with a high alignment score
- When cell type annotations or known cell type homology are unreliable, since the benchmark's quantitative scoring assumes accurate annotations and confident cell type homology
- When only species-specific genes exist with no homology graph or integrated embedding — no cross-species neighbours can be computed

## Inputs

- Integrated low-dimensional embedding or kNN graph from a cross-species integration run (e.g. SAMap or scVI output on per-species AnnData objects)
- Cell-to-species label table for every cell in the integrated object

## Outputs

- Per-strategy alignment score (average percentage of cross-species kNN neighbours per cell)
- Tabulated alignment scores (and trajectory conservation scores) per integration strategy for comparison plots

## How to apply

After running an integration strategy (SAMap via de-novo reciprocal BLAST homology, or fastMNN, Harmony, LIGER, LIGER UINMF, Scanorama, SeuratV4, scVI, scANVI using curated homologous-gene mappings), build the kNN graph in the integrated space and, for each cell, compute the percentage of its k-nearest neighbours that originate from the other species; average this over all cells to obtain the alignment score. Apply the identical kNN-based computation to every strategy so scores are comparable. Report the score alongside a biology/trajectory conservation score (e.g. stage progression conservation via BENGAL), because the benchmark showed high species mixing can coexist with poor biology conservation — SAMap achieved a significantly higher alignment score on the xenopus–zebrafish whole-body embryo task yet had a relatively low trajectory conservation score (Supplementary Fig. 19 vs Supplementary Fig. 17).

## Related tools

- **SAMap** (Produces the de-novo reciprocal BLAST homology-based whole-body integration whose kNN alignment score is computed; expected to achieve the highest alignment score) — https://github.com/atarashansky/SAMap
- **fastMNN** (Comparator integration strategy (mutual nearest neighbours) scored with the same alignment metric) — https://github.com/LTLA/batchelor
- **Harmony** (Comparator iterative-clustering integration strategy scored with the same alignment metric)
- **LIGER** (Comparator iNMF-based integration strategy scored with the same alignment metric) — https://github.com/welch-lab/liger
- **LIGER UINMF** (Comparator unshared-features iNMF integration strategy scored with the same alignment metric) — https://github.com/welch-lab/liger
- **Scanorama** (Comparator panorama-stitching integration strategy scored with the same alignment metric)
- **SeuratV4** (Comparator integration strategy scored with the same alignment metric; balances species-mixing and biology conservation) — https://github.com/satijalab/seurat-wrappers
- **scVI** (Comparator deep-generative integration strategy scored with the same alignment metric)
- **scANVI** (Comparator label-informed deep-generative integration strategy scored with the same alignment metric)
- **BENGAL** (Benchmarking pipeline in which the alignment score and companion biology conservation metric are implemented)

## Examples

```
from samap import SAMAP; sm = SAMAP(sams={'sp1': 'xenopus.h5ad', 'sp2': 'zebrafish.h5ad'}, f_maps='maps/'); sm.run()  # then compute mean fraction of cross-species kNN per cell
```

## Evaluation signals

- The alignment score must equal the mean over cells of the fraction of kNN neighbours from the other species, and the exact same kNN computation must be reused for all strategies
- On the Embryo_xt_dr xenopus–zebrafish whole-body task, SAMap should show a significantly higher alignment score than fastMNN, Harmony, LIGER, LIGER UINMF, Scanorama, SeuratV4, scVI and scANVI (Supplementary Fig. 17 pattern)
- The companion trajectory conservation score for SAMap on the embryo task should remain relatively low while its alignment score is high, reproducing the divergence between Supplementary Fig. 17 and Fig. 19
- Each per-cell cross-species neighbour percentage must lie in [0, 100] and the averaged score must be in the same range
- Scores should be reproducible under fixed random seeds for stochastic integrators (scVI, scANVI, LIGER)

## Limitations

- Quantitative scoring assumes accurate cell type annotations and confident known cell type homology from literature
- SAMap's high species mixing is heuristic and does not explain observed cell type expression similarity
- A high alignment (species-mixing) score does not imply biological fidelity: SAMap had a relatively low trajectory conservation score despite its alignment advantage

## Evidence

- [supplementary] Alignment score definition on SAMap embryo task: [evidence span withheld — anchor status: not_found]
- [abstract] SAMap advantage finding: "SAMap outperforms when integrating whole-body atlases between species with challenging gene homology annotation."
- [abstract] Balanced methods contrast: "Overall, scANVI, scVI and SeuratV4 methods achieve a balance between species-mixing and biology conservation."
- [readme] SAMap homology mechanism: [evidence span withheld — anchor status: external_doc]
- [discussion] Scoring assumption limitation: "Our quantitative scoring system of species mixing and biology conservation is supported by two assumptions: (1) cell type annotations are accurate; (2) known cell type homology from literature is"
