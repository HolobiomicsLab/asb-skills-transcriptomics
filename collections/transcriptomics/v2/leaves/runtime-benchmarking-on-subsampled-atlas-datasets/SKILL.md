---
name: runtime-benchmarking-on-subsampled-atlas-datasets
description: Use when you need to show that a single-cell integration method scales
  to atlas-level (~1 million cells, ~147 batches) cross-species data by measuring
  wall-clock runtime against deep-learning baselines at multiple subsample sizes.
when_to_use_negative:
- You only need integration-quality metrics (NMI, ARI, ASW, kBET); runtime benchmarking
  does not establish bio-conservation or batch-correction quality.
- Hardware differs across methods being compared — runtime numbers are only meaningful
  on the same GPU/CPU configuration.
- The dataset is small enough (<10k cells) that runtime differences between deep-learning
  methods are not the bottleneck.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDREAMER
  role: Method under test; trained with ADAM optimizer at batch size 128 to measure
    per-epoch runtime
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: scVI
  role: Deep-learning baseline for runtime comparison (~154 s/epoch at 1M cells)
- name: INSCT
  role: Deep-learning baseline for runtime comparison (~172 s/epoch at 1M cells)
  repo: https://github.com/lkmklsmn/insct
- name: DESC
  role: Deep-learning baseline run on the same preprocessed data
- name: iMAP
  role: Deep-learning baseline run on the same preprocessed data
- name: scANVI
  role: Deep-learning baseline run on the same preprocessed data
- name: scGEN
  role: Deep-learning baseline run on the same preprocessed data
- name: scDML
  role: Deep-learning baseline run on the same preprocessed data
- name: Scanpy
  role: Preprocessing of AnnData (normalize_total, log-transform, 2000 HVGs) so all
    methods time on identical inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:664578fcc0f9ecf8bc8875b87628f826e14c4846d2048d2136d0d8b3805da0f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - DESC
  - INSCT
  - Scanpy
  - iMAP
  - scANVI
  - scDML
  - scDREAMER
  - scGEN
  - scVI
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# runtime-benchmarking-on-subsampled-atlas-datasets

## Summary

Measures per-epoch training runtime of scDREAMER against deep-learning baselines (scVI, INSCT) on the same dataset subsampled to 10k, 100k, 500k, and 1M cells, to demonstrate scalability and the runtime advantage of the adversarial VAE over other deep-learning integration methods.

## When to use

Use when you need to show that a single-cell integration method scales to atlas-level (~1 million cells, ~147 batches) cross-species data by measuring wall-clock runtime against deep-learning baselines at multiple subsample sizes.

## When NOT to use

- You only need integration-quality metrics (NMI, ARI, ASW, kBET); runtime benchmarking does not establish bio-conservation or batch-correction quality.
- Hardware differs across methods being compared — runtime numbers are only meaningful on the same GPU/CPU configuration.
- The dataset is small enough (<10k cells) that runtime differences between deep-learning methods are not the bottleneck.

## Inputs

- Atlas-level human-mouse cross-species scRNA-seq integration dataset (~1M cells) in Scanpy AnnData format
- Datasets subsampled to 10k, 100k, 500k, and 1M cells
- Baseline method run configurations from Supplementary Table 5

## Outputs

- Runtime table in seconds per epoch for each method at each subsample size (e.g., integration_metrics_1M_cross_species.csv runtime columns)
- Comparison showing scDREAMER's runtime advantage over scVI, INSCT, and other deep-learning baselines

## How to apply

Subsample the atlas-integration dataset to 10k, 100k, 500k, and 1M cells and train each method on identical preprocessed inputs (Scanpy AnnData, normalize_total, log-transform, top 2000 highly variable genes with flavor='seurat'). For scDREAMER, train with the ADAM optimizer at batch size 128, using the scaling factor β on the KL-divergence term of the ELBO loss. Run baselines (scVI, INSCT, DESC, iMAP, scANVI, scGEN, scDML) using the configurations in Supplementary Table 5 on the same hardware — the reference measurements used one Nvidia Quadro RTX 5000 GPU. Record runtime in seconds per epoch per method per subsample size, then report the comparison (scDREAMER ~130 s/epoch vs scVI ~154 s/epoch vs INSCT ~172 s/epoch at 1M cells).

## Related tools

- **scDREAMER** (Method under test; trained with ADAM optimizer at batch size 128 to measure per-epoch runtime) — https://github.com/Zafar-Lab/scDREAMER
- **scVI** (Deep-learning baseline for runtime comparison (~154 s/epoch at 1M cells))
- **INSCT** (Deep-learning baseline for runtime comparison (~172 s/epoch at 1M cells)) — https://github.com/lkmklsmn/insct
- **DESC** (Deep-learning baseline run on the same preprocessed data)
- **iMAP** (Deep-learning baseline run on the same preprocessed data)
- **scANVI** (Deep-learning baseline run on the same preprocessed data)
- **scGEN** (Deep-learning baseline run on the same preprocessed data)
- **scDML** (Deep-learning baseline run on the same preprocessed data)
- **Scanpy** (Preprocessing of AnnData (normalize_total, log-transform, 2000 HVGs) so all methods time on identical inputs)

## Examples

```
git clone https://github.com/Zafar-Lab/scDREAMER.git; cd scDREAMER/Environments; conda env create -f scDREAMER.yml; conda activate scdreamer
```

## Evaluation signals

- Runtime table covers all four down-sampled sizes (10k, 100k, 500k, 1M) for every method compared.
- At 1M cells, scDREAMER's per-epoch runtime (~130 s) is lower than scVI (~154 s) and INSCT (~172 s) on the same GPU (Nvidia Quadro RTX 5000).
- All methods are trained on the same preprocessed data (top 2000 HVGs, flavor='seurat') so runtime differences are attributable to the method, not preprocessing.
- Runtime grows monotonically with subsample size for each method.

## Limitations

- Runtime was measured for scDREAMER against scVI and INSCT on subsampled datasets; other baselines in the 1M-cell cross-species task were evaluated for quality metrics, not necessarily timed identically.
- kBET could not be computed for some methods on the healthy heart atlas dataset due to computational limitations in handling large number of cells and batches.
- scDREAMER is a deep learning-based method and may require fine-tuning of some parameters for best performance, which can affect reported runtimes.

## Evidence

- [results] Runtime benchmarking design across subsample sizes: "we measured the runtime of scDREAMER on different datasets subsampled from the atlas-integration dataset and compared the runtime against that of scVI and INSCT"
- [results] Per-epoch runtimes at 1M cells: [evidence span withheld — anchor status: not_found]
- [methods] Training configuration: [evidence span withheld — anchor status: not_found]
- [methods] HVG preprocessing so all methods time on identical inputs: "we selected the top 2000 highly variable genes using the function “scanpy.pp.highly_variable_genes” (with flavor parameter as “seurat”)"
- [abstract] Scalability claim: "we demonstrate that scDREAMER is scalable and can perform atlas-level cross-species (e.g., human and mouse) integration while being faster than other deep-learning-based methods"
- [readme] INSCT as a baseline for large-scale integration: "`INSCT` is a deep learning algorithm which calculates an integrated embedding for scRNA-seq data. With `INSCT`, you can:

* Integrate scRNA-seq datasets across batches with/without labels."
