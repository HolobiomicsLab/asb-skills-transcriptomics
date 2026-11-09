---
name: integration-metric-visualization-across-methods
description: Use when per-cell UMAP embeddings from several integration methods are
  available together with batch and cell-type annotations, and you need to objectively
  compare how well each method mixes batches while preserving AUCell-derived cell
  types.
when_to_use_negative:
- Input lacks per-cell batch and cell-type annotations, since LISI and kBET both require
  labels for covariates (batch/donor) and cell types to score
- Only one integration embedding exists and no cross-method comparison is intended;
  single-method scores cannot be ranked against alternatives
- Cell-type labels are not trusted/conserved (e.g., random labels), making cLISI close
  to 1 unattainable and the batch-mixing vs. conservation trade-off uninterpretable
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: SingCellaR
  role: R package providing the SingCellaR object, runLISI and runKBET wrappers used
    to compute integration metrics on loaded embeddings
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: Environment in which SingCellaR, LISI and kBET computations and plots are
    executed
- name: kBET
  role: Batch-effects test whose acceptance rates, computed with a fixed seed and
    n.sample, quantify batch mixing per method
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:cfd439111564b38420a2bd9d417e200da23a3749b319cd6adad11f12d48b9912
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  - kBET
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# integration-metric-visualization-across-methods

## Summary

Quantifies and visualizes single-cell data integration quality by computing per-cell iLISI (batch mixing) and cLISI (cell-type conservation) scores plus kBET acceptance rates across multiple integration embeddings (e.g., Supervised Harmony, Harmony, Seurat RPCA, Scanorama, Limma), then aggregating them into comparison boxplots (Figure 3B/3C style).

## When to use

Use when per-cell UMAP embeddings from several integration methods are available together with batch and cell-type annotations, and you need to objectively compare how well each method mixes batches while preserving AUCell-derived cell types.

## When NOT to use

- Input lacks per-cell batch and cell-type annotations, since LISI and kBET both require labels for covariates (batch/donor) and cell types to score
- Only one integration embedding exists and no cross-method comparison is intended; single-method scores cannot be ranked against alternatives
- Cell-type labels are not trusted/conserved (e.g., random labels), making cLISI close to 1 unattainable and the batch-mixing vs. conservation trade-off uninterpretable

## Inputs

- UMAP embeddings (deposited) for each integration method: Supervised Harmony, Harmony, Seurat RPCA, Scanorama, Limma
- Per-cell batch annotation metadata (e.g., donor, batch)
- Per-cell type annotations derived from AUCell
- SingCellaR object to which embeddings and metadata are loaded

## Outputs

- Per-cell iLISI and cLISI score distributions per integration method
- kBET acceptance rates per integration method
- Figure 3B-style iLISI/cLISI comparison plot across the five methods
- Figure 3C-style kBET acceptance-rate boxplot across the five methods

## How to apply

Load the deposited UMAP embeddings for each integration method together with per-cell batch and cell-type metadata into a SingCellaR object. For each embedding, run the runLISI wrapper to compute per-cell iLISI and cLISI scores, and the runKBET wrapper after calling set.seed to fix the random seed, using the n.sample parameter to fix the number of subsampled cells so kBET scores are reproducible. Aggregate the iLISI/cLISI distributions across methods into a comparison plot (Figure 3B) and the kBET acceptance rates into a boxplot (Figure 3C). Judge the best method by high kBET and iLISI scores together with a cLISI score close to 1, as seen for Supervised Harmony on the HSPC dataset. Small run-to-run kBET variation is expected only if seed and subsample size are not fixed.

## Related tools

- **SingCellaR** (R package providing the SingCellaR object, runLISI and runKBET wrappers used to compute integration metrics on loaded embeddings) — https://github.com/supatt-lab/SingCellaR
- **R** (Environment in which SingCellaR, LISI and kBET computations and plots are executed)
- **kBET** (Batch-effects test whose acceptance rates, computed with a fixed seed and n.sample, quantify batch mixing per method)

## Examples

```
set.seed(1); runLISI(Human_HSPC, embedding_name="Supervised_Harmony_UMAP"); runKBET(Human_HSPC, n.sample=50)
```

## Evaluation signals

- Both the iLISI/cLISI comparison plot (Fig 3B) and the kBET acceptance-rate boxplot (Fig 3C) are produced for all five integration methods
- kBET acceptance rates are stable across repeated runs when set.seed and the n.sample parameter are fixed beforehand
- The best method (e.g., Supervised Harmony on HSPC data) shows high kBET and iLISI scores with cLISI close to 1
- iLISI/cLISI scores are computed per cell for every embedding loaded into the SingCellaR object, not just globally
- Slightly different kBET scores across runs indicate the seed or number of subsampling cells was not fixed and should be corrected

## Limitations

- kBET scores vary slightly between runs due to different seed number settings and the number of subsampling cells; fix set.seed and n.sample to stabilize
- UMAP/FDG plot rotations may differ from the protocol because of software version and seed number differences
- SingCellaR lacks 'automatic object transformation' to interact with other packages such as Seurat, so embeddings must be loaded appropriately into the SingCellaR object

## Evidence

- [results] Supervised Harmony shows higher kBET and iLISI scores, and cLISI score is close to 1, indicating better data integration for this HSPC dataset compared to other methods: "Supervised Harmony shows higher kBET and iLISI scores, and cLISI score is close to 1, indicating better data integration for this HSPC dataset compared to other methods"
- [discussion] fix the number of cells for the kBET analysis using the n.sample parameter described in the 'runKBET' function: [evidence span withheld — anchor status: not_found]
- [discussion] The user may encounter slightly different kBET scores from this protocol. This is due to the different seed number settings and the number of subsampling cells: "The user may encounter slightly different kBET scores from this protocol. This is due to the different seed number settings and the number of subsampling cells"
- [discussion] This issue can be solved by setting the seed number prior to running kBET using the 'set.seed' function.: [evidence span withheld — anchor status: not_found]
- [readme] an analysis package in R: "an integrative analysis tool for analysing single-cell RNA sequencing data"
