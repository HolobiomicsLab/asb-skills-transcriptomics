---
name: trajectory-conservation-scoring
description: Use when benchmarking cross-species scRNA-seq integration of whole-body
  embryonic atlases and you need to test whether stage progression / developmental
  trajectory is maintained after integration, e.g. xenopus–zebrafish Embryo_xt_dr.
when_to_use_negative:
- The datasets are tissue-specific or lung-only (e.g. Tabula Muris vs Tabula Sapiens
  Smartseq-2 lung) rather than whole-body developmental atlases with staged cells
- The goal is species mixing alone without assessing biology/stage conservation —
  the plain alignment score suffices
- Cell type annotations or stage labels are unreliable; the paper's quantitative scoring
  assumes accurate cell type annotations
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: SAMap
  role: Integration strategy using de-novo reciprocal BLAST gene-gene homology; evaluated
    for alignment score and low trajectory conservation on the embryo task
  repo: https://github.com/atarashansky/SAMap
- name: fastMNN
  role: Comparator MNN-based integration using curated homologous-gene mappings
  repo: https://github.com/LTLA/batchelor
- name: Harmony
  role: Comparator iterative-clustering-based integration using curated homologous-gene
    mappings
- name: LIGER
  role: Comparator iNMF-based integration using curated homologous-gene mappings
  repo: https://github.com/welch-lab/liger
- name: LIGER UINMF
  role: Comparator unshared-feature NMF integration using curated homologous-gene
    mappings
  repo: https://github.com/welch-lab/liger
- name: Scanorama
  role: Comparator panorama-stitching integration using curated homologous-gene mappings
- name: SeuratV4
  role: Comparator integration (via SeuratWrappers) balancing species-mixing and biology
    conservation
  repo: https://github.com/satijalab/seurat-wrappers
- name: scVI
  role: Comparator deep-learning integration balancing species-mixing and biology
    conservation
- name: scANVI
  role: Comparator label-transfer-aware deep integration balancing species-mixing
    and biology conservation
- name: BENGAL
  role: Benchmarking pipeline wrapping the alignment and trajectory conservation scoring
    of all strategies
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:f784e65ecd308bb5ec0a250c047ac25ed3484c231590a96e228565a5f04dae76
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# trajectory-conservation-scoring

## Summary

Quantify whether a cross-species integration strategy preserves developmental stage progression (trajectory conservation) when integrating whole-body embryonic atlases, as in the BENGAL benchmark's xenopus–zebrafish embryo task where SAMap scored high on alignment but low on trajectory conservation.

## When to use

Use when benchmarking cross-species scRNA-seq integration of whole-body embryonic atlases and you need to test whether stage progression / developmental trajectory is maintained after integration, e.g. xenopus–zebrafish Embryo_xt_dr.

## When NOT to use

- The datasets are tissue-specific or lung-only (e.g. Tabula Muris vs Tabula Sapiens Smartseq-2 lung) rather than whole-body developmental atlases with staged cells
- The goal is species mixing alone without assessing biology/stage conservation — the plain alignment score suffices
- Cell type annotations or stage labels are unreliable; the paper's quantitative scoring assumes accurate cell type annotations

## Inputs

- Raw count matrices for a two-species whole-body embryonic atlas (e.g. Embryo_xt_dr: xenopus and zebrafish), as per-species h5ad/AnnData objects
- Integrated embeddings from each strategy (SAMap, fastMNN, Harmony, LIGER, LIGER UINMF, Scanorama, SeuratV4, scVI, scANVI)
- Cell/stage metadata per species for trajectory (stage progression) evaluation

## Outputs

- Trajectory conservation score per integration strategy on the embryo task
- Alignment score (average percentage of cross-species kNN neighbours) per strategy
- Tabulated comparison of alignment vs trajectory conservation scores corresponding to Supplementary Fig. 17

## How to apply

Load the raw count matrices for the two-species whole-body embryo atlas (e.g. Embryo_xt_dr, xenopus and zebrafish) and prepare per-species AnnData objects. Run the integration strategy under test — e.g. SAMap (atarashansky/SAMap) with a de-novo reciprocal BLAST gene-gene homology graph, or comparators (fastMNN, Harmony, LIGER, LIGER UINMF, Scanorama, SeuratV4, scVI, scANVI) using curated homologous-gene mappings. For each integrated embedding, compute the alignment score as the average percentage of cross-species neighbours among each cell's kNN, then compute the trajectory conservation score to check whether developmental stage progression is conserved. Tabulate both scores per strategy and compare, reproducing the SAMap-versus-comparators pattern of Supplementary Fig. 17: SAMap achieves a significantly higher alignment score while retaining a low trajectory conservation score, whereas scANVI, scVI and SeuratV4 balance species-mixing and biology conservation.

## Related tools

- **SAMap** (Integration strategy using de-novo reciprocal BLAST gene-gene homology; evaluated for alignment score and low trajectory conservation on the embryo task) — https://github.com/atarashansky/SAMap
- **fastMNN** (Comparator MNN-based integration using curated homologous-gene mappings) — https://github.com/LTLA/batchelor
- **Harmony** (Comparator iterative-clustering-based integration using curated homologous-gene mappings)
- **LIGER** (Comparator iNMF-based integration using curated homologous-gene mappings) — https://github.com/welch-lab/liger
- **LIGER UINMF** (Comparator unshared-feature NMF integration using curated homologous-gene mappings) — https://github.com/welch-lab/liger
- **Scanorama** (Comparator panorama-stitching integration using curated homologous-gene mappings)
- **SeuratV4** (Comparator integration (via SeuratWrappers) balancing species-mixing and biology conservation) — https://github.com/satijalab/seurat-wrappers
- **scVI** (Comparator deep-learning integration balancing species-mixing and biology conservation)
- **scANVI** (Comparator label-transfer-aware deep integration balancing species-mixing and biology conservation)
- **BENGAL** (Benchmarking pipeline wrapping the alignment and trajectory conservation scoring of all strategies)

## Examples

```
from samap import SAMAP; sm = SAMAP(sams={'xenopus': 'Embryo_xt.h5ad', 'zebrafish': 'Embryo_dr.h5ad'}, f_maps='maps/'); sm.run()
```

## Evaluation signals

- Alignment score computed as the average percentage of cross-species neighbours among each cell's kNN in the integrated space; SAMap should be significantly higher than fastMNN, Harmony, LIGER, LIGER UINMF, Scanorama, SeuratV4, scVI, scANVI
- Trajectory conservation score for SAMap on the embryo task should remain low despite the high alignment score, matching Supplementary Fig. 17
- scANVI, scVI and SeuratV4 should show intermediate, balanced alignment and conservation scores
- Per-species AnnData inputs must be correctly normalised and the homology mapping must match the strategy type (de-novo BLAST for SAMap, curated orthology for comparators)
- Resulting score table must cover all 9 strategies × (alignment, trajectory conservation) on the Embryo_xt_dr task

## Limitations

- SAMap is heuristic and does not provide explanations of the observed cell type expression similarity
- Quantitative scoring assumes accurate cell type annotations and confident known cell type homology from literature
- Cross-species integration relies heavily on curated gene orthology based on sequence, ignoring species-specific genes (except for SAMap's de-novo homology and UINMF's unshared features)
- Expression similarities across species may arise from convergence or concerted evolution rather than homology; no algorithm disentangles these sources
- SAMap requires NCBI BLAST on PATH and the BLAST mapping step may take up to around 4 hours depending on fasta size and cores

## Evidence

- [abstract] SAMap outperforms when integrating whole-body atlases between species with challenging gene homology annotation.: "SAMap outperforms when integrating whole-body atlases between species with challenging gene homology annotation."
- [discussion] SAMap heuristic limitation: "However, being a heuristic approach, it does not provide explanations of the observed cell type expression similarity."
- [discussion] Scoring assumptions: "Our quantitative scoring system of species mixing and biology conservation is supported by two assumptions: (1) cell type annotations are accurate;"
- [abstract] scVI/scANVI/Seurat balance: "Overall, scANVI, scVI and SeuratV4 methods achieve a balance between species-mixing and biology conservation."
- [readme] SAMap BLAST requirement: [evidence span withheld — anchor status: external_doc]
