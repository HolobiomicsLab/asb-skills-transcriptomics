---
name: single-cell-gene-expression-decomposition
description: Use when seq-based ST spots (e.g., ~55 μm Visium spots) contain multiple
  cells and per-cell whole-transcriptome expressions must be reconstructed using a
  labeled scRNA-seq reference.
when_to_use_negative:
- The goal is only per-spot cell-type proportions without single-cell expression —
  deconvolution methods like RCTD, Cell2location, or CARD suffice and are cheaper
- The ST data are image-based (seqFISH/MERFISH) with single-cell resolution already;
  SpatialScope's decomposition step targets seq-based spots containing multiple cells
- No labeled scRNA-seq reference exists or platform effects cannot be corrected —
  decomposition results would not be satisfactory
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3170
tools:
- name: SpatialScope
  role: Implements the score-based generative model training (Train_scRef.py), batch-effect
    correction, and annealed Langevin dynamics posterior sampling for decomposition
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: RCTD
  role: Baseline deconvolution method for cell-type identification preceding decomposition;
    compared against SpatialScope
  repo: https://github.com/dmcable/spacexr
- name: Cell2location
  role: Baseline cell-type deconvolution method benchmarked against SpatialScope
  repo: https://github.com/BayraktarLab/cell2location
- name: CARD
  role: Baseline spatially informed deconvolution method benchmarked against SpatialScope
  repo: https://github.com/YingMa0107/CARD
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:8a5b4e04f9d7cd6e43928b3f75b54320726bea23de2a77fb98e3196dadc1c0ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - CARD
  - Cell2location
  - RCTD
  - SpatialScope
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# single-cell-gene-expression-decomposition

## Summary

Decomposes aggregated spot-level gene expression from seq-based ST data (e.g., Visium or Slide-seq spots containing multiple cells) into per-cell gene expressions at single-cell resolution by sampling from the posterior p(Xi|yi,ki) using annealed Langevin dynamics with a conditional score-based generative model trained on a scRNA-seq reference.

## When to use

Use when seq-based ST spots (e.g., ~55 μm Visium spots) contain multiple cells and per-cell whole-transcriptome expressions must be reconstructed using a labeled scRNA-seq reference.

## When NOT to use

- The goal is only per-spot cell-type proportions without single-cell expression — deconvolution methods like RCTD, Cell2location, or CARD suffice and are cheaper
- The ST data are image-based (seqFISH/MERFISH) with single-cell resolution already; SpatialScope's decomposition step targets seq-based spots containing multiple cells
- No labeled scRNA-seq reference exists or platform effects cannot be corrected — decomposition results would not be satisfactory

## Inputs

- Spot-level ST expression profile y_i from seq-based ST data (Visium or Slide-seq) with assigned cell-type labels k_i
- Cell-type-labeled scRNA-seq reference (.h5ad, e.g., Ref_Heart_sanger_D2.h5ad) normalized and restricted to HVGs plus marker genes
- Trained conditional score model checkpoint (e.g., model_5000.pt) with per-cell-type mean expression vectors mu_k
- Batch-effect correction factors exp(gamma_g_hat) from ST-to-reference platform-effect adjustment

## Outputs

- Posterior-mean per-cell gene expression vectors x_i,m (single-cell resolution decomposition of each spot)
- Validation metrics (cosine similarity between decomposed and ground-truth simulated per-cell expressions) written to decomposition_validation.csv

## How to apply

First prepare the scRNA-seq reference (e.g., Ref_Heart_sanger_D2.h5ad) with total-count normalization, log(1+x) transformation, per-cell-type mean subtraction (mu_k), and selection of top 1000 highly variable genes plus 50 marker genes per cell type. Train the conditional score network s_theta(x, sigma_l, mu_k) by denoising score matching over L Gaussian noise levels (sigma_begin=50, sigma_end=0.002, step_lr=3e-7) using Train_scRef.py, saving checkpoints such as model_5000.pt. Correct batch effects on the spot-level profile y_i by dividing each gene count by exp(gamma_g_hat) and normalizing the total count to the mean per-cell total from the reference; this is essential because decomposition fails if platform effects are not corrected. Run annealed Langevin dynamics (Algorithm 1) with X(t+1) = X(t) + eta*[grad_X log p(y|X) + s_theta(x_i,m, sigma_l, mu_k_i,m)] + sqrt(2*eta)*eps, using step size eta = eta0 * sigma_l^2 / sigma_1^2, L=232 noise levels, T=5 steps per level, and sigma_yl = sqrt(sigma_l), initializing each level from the previous one and averaging final samples over R repeats to obtain posterior means. For Slide-seq data, replace nucleus segmentation with singlet/doublet classification (at most two cell types per spot) and decompose simulated two-cell spots y = x1 + x2 + noise under the forward model y_i | X_i ~ N(f(X_i), sigma_y^2 I) with f = sum_m (exp(x_i,m + mu_k_i,m) - 1).

## Related tools

- **SpatialScope** (Implements the score-based generative model training (Train_scRef.py), batch-effect correction, and annealed Langevin dynamics posterior sampling for decomposition) — https://github.com/YangLabHKUST/SpatialScope
- **RCTD** (Baseline deconvolution method for cell-type identification preceding decomposition; compared against SpatialScope) — https://github.com/dmcable/spacexr
- **Cell2location** (Baseline cell-type deconvolution method benchmarked against SpatialScope) — https://github.com/BayraktarLab/cell2location
- **CARD** (Baseline spatially informed deconvolution method benchmarked against SpatialScope) — https://github.com/YingMa0107/CARD

## Examples

```
python Train_scRef.py --sigma_begin 50 --sigma_end 0.002 --step_lr 3e-7  # then load model_5000.pt and run annealed Langevin posterior sampling per spot
```

## Evaluation signals

- High cosine similarity between posterior-mean decomposed expressions x1, x2 and the ground-truth simulated per-cell expressions — the metric the paper uses to benchmark decomposition performance
- The trained checkpoint (e.g., model_5000.pt) loads successfully and supports posterior sampling via Langevin dynamics
- Simulated two-cell spot with known ground truth (y = x1 + x2 + noise under the forward model f = sum_m (exp(x_i,m + mu_k_i,m) - 1)) is decomposed accurately
- Per-cell total counts after batch correction match the mean per-cell total from the reference
- Metrics are written to decomposition_validation.csv with expected schema

## Limitations

- Training the deep generative models is time-consuming, measured in hours or days
- Decomposition results are unsatisfactory if platform effects between ST and scRNA-seq data are not appropriately corrected
- For Slide-seq, singlet/doublet classification assumes at most two cell types co-exist within a spot

## Evidence

- [methods] we use Langevin dynamics to get samples from the posterior p(Xi|yi,ki): [evidence span withheld — anchor status: not_found]
- [methods] batch effect correction before decomposition: "we adjust the batch effects between ST and single-cell reference data before we perform gene expression decomposition"
- [discussion] platform effects must be corrected: "Cell type identification and gene expression decomposition results would not be satisfactory if the platform effects are not appropriately corrected."
- [methods] decomposition step definition: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [methods] Slide-seq singlet/doublet assumption: "we replace the first step nucleus segmentation by singlet/doublet classification, which assumes that at most two cell types co-exist within a Slide-seq spot"
- [intro] Visium spot size motivation: "considering the larger spot size of 55 μm, a Visium spot often contains multiple cells, which limits its usage in resolving detailed tissue structure"
- [readme] repo installation: "$ git clone https://github.com/YangLabHKUST/SpatialScope.git"
