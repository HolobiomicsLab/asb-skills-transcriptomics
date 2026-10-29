---
name: scrnaseq-dataset-subsetting
description: Use when benchmarking PAS/scoring tools on million-cell-level scRNA-seq
  data where full-dataset computation is impractical, and a fixed-size random cell
  subset (e.g., 100k cells from the >1.2M-cell lupus atlas) suffices for validation.
when_to_use_negative:
- The downstream goal is full-scale multicellular pathway module discovery with MOFA
  — that requires all samples/cells and studies with at least 15 samples, not a 100k-cell
  benchmark subset
- The dataset is already small enough that direct full-data tool comparison or scoring
  is computationally feasible
- The analysis depends on precise population composition where a random subsample
  could distort rare-cell-type representation (ranking-based fast_ucell PAS is composition-insensitive,
  but cell-type proportions are not)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPAFA
  role: Provides the 'fast_Ucell' PAS implementation run on the 100k-cell subsets
  repo: github.com/ZhuoliHuang/scPAFA
- name: AUCell
  role: Reference PAS method computed on the same subsets for per-cell cosine-similarity
    comparison
- name: UCell
  role: Original scoring method that fast_Ucell reimplements efficiently; same max
    rank threshold ties it to AUCell
- name: scikit-learn
  role: Computes per-cell cosine similarity between fast_ucell and AUCell PAS vectors
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:86fabbc1a5574e65030eb5d44e0f44c774f235f515475e162b341805f0f54a48
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AUCell
  - Python
  - UCell
  - scPAFA
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# scrnaseq-dataset-subsetting

## Summary

Draws fixed-size cell subsets (e.g., 100,000 cells) from million-cell-scale scRNA-seq atlases such as the >1.2M-cell lupus PBMC atlas and CRC tumor datasets, so that method-comparison benchmarks like the fast_ucell-vs-AUCell PAS cosine-similarity validation can run in tractable time.

## When to use

Use when benchmarking PAS/scoring tools on million-cell-level scRNA-seq data where full-dataset computation is impractical, and a fixed-size random cell subset (e.g., 100k cells from the >1.2M-cell lupus atlas) suffices for validation.

## When NOT to use

- The downstream goal is full-scale multicellular pathway module discovery with MOFA — that requires all samples/cells and studies with at least 15 samples, not a 100k-cell benchmark subset
- The dataset is already small enough that direct full-data tool comparison or scoring is computationally feasible
- The analysis depends on precise population composition where a random subsample could distort rare-cell-type representation (ranking-based fast_ucell PAS is composition-insensitive, but cell-type proportions are not)

## Inputs

- Full scRNA-seq expression matrix (e.g., CRC datasets and >1.2M-cell lupus PBMC atlas, SLE vs healthy)
- Target subset size (100,000 cells in the scPAFA benchmark)
- Pathway dictionary (pathway name -> gene list) from MsigDB or NCATS BioPlanet

## Outputs

- 100k-cell expression matrix subset per dataset (CRC subset, lupus subset)
- Subset-compatible PAS matrices (e.g., 1629 pathways for CRC, 1383 for lupus) computed on the sampled cells

## How to apply

Load the full scRNA-seq expression matrix (AnnData-style) for the target study — here the CRC datasets and the >1.2M-cell lupus PBMC atlas — and randomly sample a fixed number of cells (100,000 in the scPAFA benchmark) to form a subset per dataset. Keep the accompanying pathway dictionaries (MsigDB or NCATS BioPlanet; 1629 pathways for CRC, 1383 for lupus) aligned with the subset. Compute PAS on the subset with scPAFA's 'fast_Ucell' (and a reference implementation such as AUCell using the same max rank threshold); because fast_ucell PAS is based on within-cell expression ranking, it is 'insensitive to the dataset in which the cells are located', so subset-based agreement statistics transfer to the full data. Subsetting is a benchmarking convenience only — the original scPAFA claim is that PAS for 1383 pathways can be computed on the full million-cell data within 30 minutes.

## Related tools

- **scPAFA** (Provides the 'fast_Ucell' PAS implementation run on the 100k-cell subsets) — github.com/ZhuoliHuang/scPAFA
- **AUCell** (Reference PAS method computed on the same subsets for per-cell cosine-similarity comparison)
- **UCell** (Original scoring method that fast_Ucell reimplements efficiently; same max rank threshold ties it to AUCell)
- **scikit-learn** (Computes per-cell cosine similarity between fast_ucell and AUCell PAS vectors)

## Examples

```
import numpy as np; sub = np.random.choice(adata.n_obs, size=100000, replace=False); adata_sub = adata[sub].copy()
```

## Evaluation signals

- Each subset contains exactly the target number of cells (100,000) sampled without replacement from the parent atlas
- Average per-cell cosine similarity between fast_ucell and AUCell PAS on the CRC subset reproduces the reported 0.97 (Fig. 1d) within a small tolerance
- Average per-cell cosine similarity on the lupus subset reproduces the reported 0.94 within tolerance
- Results are written to pas_cosine_similarity_comparison.csv with the CRC and lupus averages
- Pathway counts match the parent dictionaries (1629 pathways for CRC, 1383 for lupus) — no pathways lost during subsetting

## Limitations

- Subsetting is only a benchmarking proxy: the cosine-similarity validation (0.97 CRC / 0.94 lupus) is established on 100k-cell subsets, not the full >1.2M-cell atlas
- Fast_ucell PAS is ranking-based within each cell, so it is insensitive to dataset composition — but this property applies to the PAS, not to cell-type composition of the subset itself
- The article reports that methods like AddModuleScore, UCell, and AUCell exhibit 'tardiness and computational inefficacy in large-scale scRNA-seq datasets, especially in situations where a large number of pathways (>10^3) is required', which is precisely why subsetting was needed for the comparison

## Evidence

- [intro] which can compute PAS for 1383 pathways in million-cell-level scRNA-seq data within 30 min: "which can compute PAS for 1383 pathways in million-cell-level scRNA-seq data within 30 min"
- [intro] benchmark on CRC and lupus datasets: "we benchmarked scPAFA’s performance on colorectal cancer (CRC) datasets22,23 together with a large-scale lupus atlas3"
- [discussion] fast_ucell matches AUCell via max rank threshold: "The “fast_ucell” function can provide the same PAS of UCell, which is closely related to AUCell PAS by setting the same max rank threshold."
- [discussion] ranking-based PAS insensitive to dataset composition: "Similar to AUCell, the PAS of “fast_ucell” is based on the ranking of gene expression within each cell, making them insensitive to the dataset in which the cells are located."
- [abstract] large-scale lupus atlas scale: "large-scale lupus atlas over 1.2 million cells"
