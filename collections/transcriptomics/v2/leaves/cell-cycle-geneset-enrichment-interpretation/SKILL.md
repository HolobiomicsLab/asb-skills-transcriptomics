---
name: cell-cycle-geneset-enrichment-interpretation
description: Use when you need to show that periodic genes discovered ab initio from
  a circular manifold (Cyclum pseudo-time) are more enriched for GO:0007049 cell-cycle
  terms than genes ranked by principal components on scRNA-seq data.
when_to_use_negative:
- The biological process is a linear trajectory rather than circular/periodic — Cyclum
  fits predefined circular manifolds, so pseudo-time and gene rankings are not appropriate
- You already have validated cell-cycle phase labels and only need to refine them
  along pseudo-time — use cyclum.postproc.refine_labels instead of an ab-initio ranking
  + GSEA comparison
- The analysis relies on marker-gene-based phase scoring (e.g., scanpy.tl.score_genes_cell_cycle
  or Seurat::CellCycleScoring) — Cyclum's methodology relies on the circular manifold
  instead of the marker genes
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Cyclum
  role: Sinusoidal autoencoder that projects mESC expression onto a circular manifold
    to infer circular pseudo-time and rank genes by match with the inferred periodicity
  repo: https://github.com/KChen-lab/Cyclum
- name: GSEA
  role: Broad-institute enrichment tool run on each ranked gene list against the GO:0007049
    cell-cycle set to obtain NES values
  repo: http://software.broadinstitute.org/gsea/index.jsp
- name: Cyclum mESC notebooks (cyclum2)
  role: Reference notebook reproducing the Cyclum analysis of the mESC dataset used
    to build the ranked lists
  repo: https://github.com/lshh125/cyclum2/blob/master/tests/notebooks/mESC.ipynb
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:4afbe21480b48abe03e2bd003b9c027bc898861bb2e431dfa4833b5f39add961
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum
  tools:
  - Cyclum
  - Cyclum mESC notebooks (cyclum2)
  - GSEA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cell-cycle-geneset-enrichment-interpretation

## Summary

Quantify whether cell-cycle genes ranked ab initio from a circular manifold (Cyclum) better capture cell-cycle biology than linear PCA rankings, by running GSEA against the GO:0007049 cell-cycle gene set and comparing normalized enrichment scores (Cyclum NES 1.57 vs. PC1 1.07 / PC2 1.06 on mESC).

## When to use

Use when you need to show that periodic genes discovered ab initio from a circular manifold (Cyclum pseudo-time) are more enriched for GO:0007049 cell-cycle terms than genes ranked by principal components on scRNA-seq data.

## When NOT to use

- The biological process is a linear trajectory rather than circular/periodic — Cyclum fits predefined circular manifolds, so pseudo-time and gene rankings are not appropriate
- You already have validated cell-cycle phase labels and only need to refine them along pseudo-time — use cyclum.postproc.refine_labels instead of an ab-initio ranking + GSEA comparison
- The analysis relies on marker-gene-based phase scoring (e.g., scanpy.tl.score_genes_cell_cycle or Seurat::CellCycleScoring) — Cyclum's methodology relies on the circular manifold instead of the marker genes

## Inputs

- mESC scRNA-seq expression matrix (cells x genes)
- GO:0007049 cell-cycle biological-process gene set (GMT)
- Gene ranked lists from three orderings: Cyclum periodicity-match score, PCA component 1, PCA components 1-2
- GSEA-compatible files: expression .txt and continuous phenotype .cls (via mat2txt / vec2cls / mat2cls)

## Outputs

- gsea_nes_results.csv with one normalized enrichment score per ranking (Cyclum, PC1, PC1-2)
- GSEA-compatible .txt expression and .cls phenotype files for each ranked list

## How to apply

Preprocess the mESC scRNA-seq matrix with a log2(x+1) transformation, then scale to variance = 1 and center to mean = 1, which preserves the circular pattern for Cyclum. Fit the Cyclum sinusoidal autoencoder to project cells onto a circular manifold and obtain the circular pseudo-time, then rank genes ab initio by the degree of match between each gene's expression kinetics and the inferred periodicity. Independently run PCA on the same data and build two baseline ranked lists from component 1 and components 1-2. Convert each ranked list to GSEA format (the Cyclum repo provides R helpers mat2txt, vec2cls and mat2cls for expression matrices and continuous phenotypes) and run GSEA against the GO:0007049 cell-cycle gene set for all three rankings. Collect the normalized enrichment scores into gsea_nes_results.csv and interpret: the Cyclum-based ranking should yield the highest NES (reported 1.57) versus PCA-based NES values of 1.07 and 1.06, indicating the circular representation captures cell-cycle periodicity that linear components miss.

## Related tools

- **Cyclum** (Sinusoidal autoencoder that projects mESC expression onto a circular manifold to infer circular pseudo-time and rank genes by match with the inferred periodicity) — https://github.com/KChen-lab/Cyclum
- **GSEA** (Broad-institute enrichment tool run on each ranked gene list against the GO:0007049 cell-cycle set to obtain NES values) — http://software.broadinstitute.org/gsea/index.jsp
- **Cyclum mESC notebooks (cyclum2)** (Reference notebook reproducing the Cyclum analysis of the mESC dataset used to build the ranked lists) — https://github.com/lshh125/cyclum2/blob/master/tests/notebooks/mESC.ipynb

## Examples

```
git clone https://github.com/lshh125/cyclum2 && jupyter notebook tests/notebooks/mESC.ipynb
```

## Evaluation signals

- The Cyclum-based ranking must produce the highest NES of the three lists (reported value 1.57)
- PCA-based NES values should reproduce the reported 1.07 (PC1) and 1.06 (PC2), with all three NES positive for the cell-cycle set
- gsea_nes_results.csv contains exactly three NES entries, one per ranking (Cyclum, PC1, PC1-2)
- Preprocessing invariant: input matrix is log2(x+1) transformed, scaled to variance = 1 and centered to mean = 1 before fitting
- Each ranked list is emitted in GSEA-compatible .txt/.cls format before running GSEA

## Limitations

- Cyclum requires TensorFlow 1.x and tested package versions (python 3.7.4, keras 2.2.4, tensorflow 1.14.0, numpy 1.16.5); installing latest dependencies can break the run
- The paper does not state Python/Keras/TensorFlow version numbers, and hardware/runtime details (GTX 960M GPU, 2 GB graphics memory, 40 min for 12,280 cells x 33,694 genes) are only in Supplementary Note 2
- Reproducing NES 1.57 vs. 1.07/1.06 depends on the exact preprocessing (scaled, centered log2(x+1)) and on ranking genes by degree of match with the inferred periodicity; deviations change the ordering and NES values

## Evidence

- [methods] scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well.: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [intro] Cyclum can be used to identify genes associated with the periodic process, based on the degrees of match between the kinetics of gene expressions and the inferred periodicity.: "Cyclum can be used to identify genes associated with the periodic process, based on the degrees of match between the kinetics of gene expressions and the inferred periodicity."
- [abstract] we develop Cyclum, an autoencoder approach identifying circular trajectories in the gene expression space: "we develop Cyclum, an autoencoder approach identifying circular trajectories in the gene expression space"
- [readme] `vec2cls`, which writes phenotypes (either discrete, e.g., cell type, or continuous, e.g., pseudotime) to a GSEA compatible `.cls` file,: [evidence span withheld — anchor status: external_doc]
- [methods] `Cyclum analysis of the mESC dataset <https://github.com/lshh125/cyclum2/blob/master/tests/notebooks/mESC.ipynb>`_: "`Cyclum analysis of the mESC dataset <https://github.com/lshh125/cyclum2/blob/master/tests/notebooks/mESC.ipynb>`_"
