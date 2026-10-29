---
name: celltype-label-matching-evaluation
description: Use when you have aligned single-cell embeddings from two modalities
  (e.g., scRNA-seq and scATAC-seq) with cell-type labels available post hoc, and need
  to quantify alignment quality across competing methods.
when_to_use_negative:
- The modalities have not yet been aligned — run the alignment methods first; this
  skill only scores existing aligned joint embeddings
- The dataset lacks cell-type or subcell-type labels for at least one modality, since
  Celltype/Subcelltype matching cannot be computed without them
- You need a fully unsupervised quality metric on unlabeled data — use batch LISI
  alone (optimal value 2) rather than label-matching scores
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scTopoGAN
  role: Produces the aligned embeddings whose Celltype/Subcelltype matching and batch
    LISI are scored; run via run_scTopoGAN
  repo: github.com/AkashCiel/scTopoGAN
- name: UnionCom
  role: Baseline method whose aligned embeddings are evaluated with the same label-matching
    and LISI metrics
  repo: github.com/AkashCiel/scTopoGAN
- name: MMD-MA
  role: Baseline method applied via manifold_align_mmd_pytorch.py and evaluated with
    the same metrics
  repo: github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:756ac45955e214e51367f7de8bbc0f76d9020819003a3e4aa12ba3655a5be418
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MMD-MA
  - SCOTv2
  - UnionCom
  - scTopoGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# celltype-label-matching-evaluation

## Summary

Evaluates unsupervised single-cell manifold alignments (e.g., scTopoGAN vs UnionCom, MMD-MA, SCOTv2) by scoring Celltype and Subcelltype label matching on the aligned joint embeddings, complemented by batch LISI as a modality-mixing metric.

## When to use

Use when you have aligned single-cell embeddings from two modalities (e.g., scRNA-seq and scATAC-seq) with cell-type labels available post hoc, and need to quantify alignment quality across competing methods.

## When NOT to use

- The modalities have not yet been aligned — run the alignment methods first; this skill only scores existing aligned joint embeddings
- The dataset lacks cell-type or subcell-type labels for at least one modality, since Celltype/Subcelltype matching cannot be computed without them
- You need a fully unsupervised quality metric on unlabeled data — use batch LISI alone (optimal value 2) rather than label-matching scores

## Inputs

- Aligned joint embeddings (.npy) of source and target single-cell modalities per method-dataset pair
- Cell-type labels for the aligned cells (e.g., Celltype and Subcelltype annotations for Full PBMC, Partial PBMC, BM datasets)
- Per-method aligned outputs from scTopoGAN (run_scTopoGAN), UnionCom, MMD-MA, and SCOTv2

## Outputs

- CSV with one Celltype matching, Subcelltype matching, and batch LISI entry per method-dataset combination (Table-2 equivalent)
- Aligned embedding .npy files per method and dataset
- Comparison table of mean ± standard deviation label-matching scores across GAN runs (e.g., 40 GANs)

## How to apply

After each method (scTopoGAN via run_scTopoGAN, UnionCom via UnionCom.py, MMD-MA via manifold_align_mmd_pytorch.py, SCOTv2) aligns the source manifold to the target manifold, compute Celltype matching, Subcelltype matching, and batch LISI on the aligned joint embeddings for every method-dataset pair (Full PBMC, Partial PBMC, BM, scGEM). Batch LISI has an optimal value of 2, so scores approaching 2 indicate superior modality mixing; higher Celltype/Subcelltype matching percentages indicate better label agreement after alignment. Aggregate one value per metric per method-dataset combination into a single Table-2-equivalent CSV and save the aligned embeddings as .npy files. Report label-matching scores as mean ± standard deviation across GAN runs when evaluating stochastic methods (scTopoGAN's Celltype matching over 40 GANs was 37.1 ± 16.5%), since alignment quality varies widely across initializations despite similar losses.

## Related tools

- **scTopoGAN** (Produces the aligned embeddings whose Celltype/Subcelltype matching and batch LISI are scored; run via run_scTopoGAN) — github.com/AkashCiel/scTopoGAN
- **UnionCom** (Baseline method whose aligned embeddings are evaluated with the same label-matching and LISI metrics) — github.com/AkashCiel/scTopoGAN
- **MMD-MA** (Baseline method applied via manifold_align_mmd_pytorch.py and evaluated with the same metrics) — github.com/AkashCiel/scTopoGAN

## Examples

```
from scTopoGAN_Functions import run_scTopoGAN; aligned = run_scTopoGAN(source_manifold, target_manifold)  # then compute Celltype matching, Subcelltype matching, batch LISI on aligned embeddings
```

## Evaluation signals

- The metrics CSV contains exactly one Celltype matching, Subcelltype matching, and batch LISI entry per method-dataset combination
- Batch LISI values fall in the range up to the optimal value 2; scTopoGAN should reach the highest batch LISI across Full PBMC, Partial PBMC, and BM
- Values reproduce the corresponding Table 2 results from the scTopoGAN preprint
- Stochastic GAN evaluations report mean ± standard deviation across GAN runs (baseline variability ~37.1 ± 16.5% Celltype matching over 40 GANs), not a single-point score
- Aligned embeddings are saved as .npy files for each method and dataset, enabling downstream recomputation of the metrics

## Limitations

- Label matching requires ground-truth Celltype/Subcelltype annotations, so it cannot serve as the model-selection criterion inside the unsupervised scTopoGAN loop
- Alignment quality varies widely across differently initialized GANs despite similar losses (37.1 ± 16.5% Celltype matching over 40 GANs), so single-run label-matching scores are unreliable without replicates
- Alignment performance degrades when the ATAC modality is downsampled: stratified Celltype matching drops at ~30% removal and becomes more variable at ≥60% cell-type-specific removal

## Evidence

- [methods] ```run_scTopoGAN``` which aligns the source manifold to the target manifold using the topology-guided GAN model: "```run_scTopoGAN``` which aligns the source manifold to the target manifold using the topology-guided GAN model"
- [abstract] scTopoGAN outperforms state-of-the-art manifold alignment methods in complete unsupervised settings: "We show that scTopoGAN outperforms state-of-the-art manifold alignment methods in complete unsupervised settings."
- [readme] The ```UnionCom.py``` and ```manifold_align_mmd_pytorch.py``` scripts are used to apply UnionCom and MMD-MA, respectively.: "The ```UnionCom.py``` and ```manifold_align_mmd_pytorch.py``` scripts are used to apply UnionCom and MMD-MA, respectively."
