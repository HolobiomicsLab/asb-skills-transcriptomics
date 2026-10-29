---
name: gmm-based-pseudo-time-reconstruction
description: Use when benchmarking cell-cycle pseudo-time inference on scRNA-seq data
  — e.g., five melanoma patient 10X Chromium samples (Mel78–Mel88) — and a model-free
  GMM baseline ordering is needed for GSEA NES comparison against a circular model
  such as Cyclum.
when_to_use_negative:
- The latent process is not circular/periodic — linear-representation trajectory methods
  or PCA-based orderings, not GMM-based cell-cycle reconstruction, are appropriate
  for non-periodic pseudo-time
- 'Goal is accurate cell-cycle pseudo-time itself rather than benchmarking: the paper''s
  finding is that reCAT constitutes a disadvantage versus Cyclum (''a clear advantage
  over other model-free approaches, such as reCAT''), so reCAT ordering alone should
  not be the final result'
- Phase labels already exist from marker-gene scoring (e.g., scanpy score_genes_cell_cycle
  or Seurat CellCycleScoring) and only need refinement — Cyclum's refine_labels workflow
  applies instead of re-running GMM reconstruction
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: reCAT
  role: Reconstructs cell-cycle pseudo-time using a Gaussian mixture model on the
    preprocessed expression matrices; serves as the model-free baseline in the GSEA
    NES comparison
- name: Cyclum
  role: Comparison method whose sinusoidal autoencoder pseudo-time is ranked against
    the reCAT GMM ordering; also defines the required preprocessing (log2(x+1), centering,
    scaling)
  repo: https://github.com/KChen-lab/Cyclum
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:0e5d434284f0cae2fa7414b8eba7dddad03f40875458d34feb631716b886bbf1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum
  tools:
  - Cyclum
  - reCAT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# gmm-based-pseudo-time-reconstruction

## Summary

Reconstruct single-cell cell-cycle pseudo-time with reCAT's Gaussian mixture model (GMM) on log2(x+1) transformed, centered (mean=1), scaled (variance=1) scRNA-seq matrices, and use the resulting orderings as a model-free baseline benchmarked against Cyclum, PC1, and PC2 via GSEA enrichment of the GO cell-cycle gene set.

## When to use

Use when benchmarking cell-cycle pseudo-time inference on scRNA-seq data — e.g., five melanoma patient 10X Chromium samples (Mel78–Mel88) — and a model-free GMM baseline ordering is needed for GSEA NES comparison against a circular model such as Cyclum.

## When NOT to use

- The latent process is not circular/periodic — linear-representation trajectory methods or PCA-based orderings, not GMM-based cell-cycle reconstruction, are appropriate for non-periodic pseudo-time
- Goal is accurate cell-cycle pseudo-time itself rather than benchmarking: the paper's finding is that reCAT constitutes a disadvantage versus Cyclum ('a clear advantage over other model-free approaches, such as reCAT'), so reCAT ordering alone should not be the final result
- Phase labels already exist from marker-gene scoring (e.g., scanpy score_genes_cell_cycle or Seurat CellCycleScoring) and only need refinement — Cyclum's refine_labels workflow applies instead of re-running GMM reconstruction

## Inputs

- 10X Chromium single-cell RNA-seq count matrices for melanoma patient samples (Mel78, Mel79, Mel80, Mel81, Mel88)
- Preprocessed expression matrices: log2(x+1) transformed, centered (mean=1), scaled (variance=1)
- GO cell-cycle gene set for GSEA

## Outputs

- Per-cell cell-cycle pseudo-time ordering from reCAT's Gaussian mixture model
- GSEA normalized enrichment scores (NES) for the cell-cycle gene set per method per sample
- Ranking table of Cyclum, reCAT, PC1, and PC2 by NES (e.g., gsea_nes_ranking.csv)

## How to apply

Preprocess each 10X Chromium count matrix with a log2(x+1) transform followed by centering to mean 1 and scaling to variance 1, the input format both reCAT and Cyclum expect. Run reCAT, which reconstructs cell-cycle pseudo-time with a Gaussian mixture model, on the same preprocessed matrices to obtain a per-sample cell ordering. As linear-representation baselines for the same comparison, compute PC1 and PC2 orderings via PCA. For each method (reCAT, Cyclum, PC1, PC2) and each sample, run GSEA using the inferred pseudo-time ordering against the GO cell-cycle gene set and record the normalized enrichment score (NES). Rank the methods by NES per sample (as in Fig. 4a / Supplementary Table 1); on the five melanoma samples, Cyclum's pseudo-time received the highest GSEA cell-cycle enrichment scores, outperforming reCAT, PC1, and PC2 — including on the low-cycling sample Mel79 — which sets the expected validation outcome.

## Related tools

- **reCAT** (Reconstructs cell-cycle pseudo-time using a Gaussian mixture model on the preprocessed expression matrices; serves as the model-free baseline in the GSEA NES comparison)
- **Cyclum** (Comparison method whose sinusoidal autoencoder pseudo-time is ranked against the reCAT GMM ordering; also defines the required preprocessing (log2(x+1), centering, scaling)) — https://github.com/KChen-lab/Cyclum

## Evaluation signals

- Cyclum attains the highest cell-cycle GSEA NES across all five melanoma samples, including the low-cycling sample Mel79, matching the ranking in Fig. 4a / Supplementary Table 1
- reCAT runs on the identical preprocessed matrices (log2(x+1), mean=1, variance=1) used for Cyclum and PCA, so NES differences are attributable to the ordering method
- The output ranking table contains an NES per method (Cyclum, reCAT, PC1, PC2) per sample (Mel78, Mel79, Mel80, Mel81, Mel88)
- reCAT ordering reproduces the expected behavior of a model-free GMM baseline: NES at or below Cyclum's on each sample
- GSEA input files are in Broad-compatible format (expression .txt plus continuous phenotype .cls written via the repo's mat2txt/vec2cls helpers)

## Limitations

- reCAT is a model-free approach and constitutes a disadvantage relative to Cyclum's model-based circular manifold fitting for cell-cycle characterization
- Cyclum's model-based design (fitting data to predefined circular manifolds) is what makes it robust to random noise and small sample sizes — a limitation the GMM baseline shares less well
- GSEA evaluation depends on the ordering quality of the baseline; linear PC1/PC2 orderings cannot accurately represent circular timings, and reCAT's GMM ordering likewise cannot unfold the circular manifold

## Evidence

- [intro] reCAT reconstructs cell-cycle pseudo-time using a GMM: "A recent method reCAT25 reconstructs cell-cycle pseudo-time using a Gaussian mixture model (GMM)"
- [discussion] Cyclum advantage over reCAT: "It constitutes a clear advantage over other model-free approaches, such as reCAT, for the purpose of characterizing cell cycles."
- [methods] Preprocessing requirement: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [readme] GSEA export from README: [evidence span withheld — anchor status: external_doc]
