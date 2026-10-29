---
name: scrna-seq-batch-integration-benchmarking
description: Use when integrating multiple scRNA-seq batches or species (e.g., human–mouse
  atlas data up to ~1M cells) and you need to compare integration methods' batch mixing,
  cell-type conservation, and runtime.
when_to_use_negative:
- The goal is supervised label prediction or label transfer rather than benchmarking
  integration quality — use scDREAMER-Sup-style semi-supervised evaluation instead
- Data have not been quality-controlled or reduced to shared highly variable genes;
  running integrators on raw, unnormalized counts without the common preprocessing
  will invalidate cross-method comparison
- Datasets are so large that a metric like kBET cannot be computed within available
  memory — as in the healthy heart atlas where 'kBET could not be computed for some
  of the methods due to computational limitations'
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDREAMER
  role: 'Primary integration method under benchmark: adversarial variational autoencoder
    trained with ADAM (batch size 128) producing batch-mixed latent embeddings'
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: scVI
  role: Deep generative (VAE) baseline integration method; also a runtime comparison
    point (~154 s/epoch vs. ~130 s/epoch for scDREAMER)
- name: DESC
  role: Deep-learning baseline integration method run on the same preprocessed data
- name: iMAP
  role: GAN + autoencoder baseline integration method run on the same preprocessed
    data
- name: scANVI
  role: Deep-learning baseline integration method run on the same preprocessed data
- name: scGEN
  role: Deep-learning baseline integration method run on the same preprocessed data
- name: scDML
  role: Deep-learning baseline integration method run on the same preprocessed data
- name: INSCT
  role: Batch-aware triplet-network integration method used as a runtime baseline
    (~172 s/epoch on the 1M-cell cross-species dataset)
  repo: https://github.com/lkmklsmn/insct
- name: Scanpy
  role: Preprocessing (normalize_total, log-transform, highly_variable_genes flavor='seurat',
    top 2000 HVGs) and AnnData data container
- name: Louvain clustering
  role: Clusters each integrated embedding at the resolution maximizing NMI to compute
    bio-conservation metrics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:47c0d1c804b4696ae4e71e9078c9a7fd0badfc1ba81afe1d048c6b16ffe4d5b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - DESC
  - INSCT
  - Louvain clustering
  - Scanpy
  - iMAP
  - scANVI
  - scDML
  - scDREAMER
  - scGEN
  - scVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# scrna-seq-batch-integration-benchmarking

## Summary

Benchmark single-cell RNA-seq batch-integration methods (e.g., scDREAMER vs. scVI, DESC, iMAP, scANVI, scGEN, scDML, INSCT) on shared preprocessed data, scoring batch correction and bio-conservation with NMI, ARI, ASW, PCR batch, graph connectivity, and kBET, plus runtime scaling up to ~1M cells.

## When to use

Use when integrating multiple scRNA-seq batches or species (e.g., human–mouse atlas data up to ~1M cells) and you need to compare integration methods' batch mixing, cell-type conservation, and runtime.

## When NOT to use

- The goal is supervised label prediction or label transfer rather than benchmarking integration quality — use scDREAMER-Sup-style semi-supervised evaluation instead
- Data have not been quality-controlled or reduced to shared highly variable genes; running integrators on raw, unnormalized counts without the common preprocessing will invalidate cross-method comparison
- Datasets are so large that a metric like kBET cannot be computed within available memory — as in the healthy heart atlas where 'kBET could not be computed for some of the methods due to computational limitations'

## Inputs

- Raw count expression matrices as Scanpy AnnData objects (.h5ad) for each batch/dataset
- Batch labels (and optionally cell-type annotations for supervised variants like scDREAMER-Sup)
- Run configurations for competing methods (Supplementary Table 5)
- Benchmarking datasets: human pancreas (16,382 cells), lung atlas (32,472 cells), human immune (33,506 cells), heart atlas (486,134 cells), macaque retina, and ~1M-cell human–mouse cross-species data

## Outputs

- Batch-mixed latent embeddings per method
- Louvain cluster assignments at NMI-maximizing resolution
- Integration metric table: NMI, ARI, cell-type ASW, batch ASW, PCR batch, graph connectivity, kBET (k=50)
- Runtime table (seconds) at 10k/100k/500k/1M-cell subsamples
- Composite bio-conservation, batch-correction, and combined composite scores (min–max normalized to [0,1]), e.g., integration_metrics_1M_cross_species.csv

## How to apply

Preprocess every dataset identically: import raw counts as a Scanpy AnnData object, remove low-quality cells based on mitochondrial gene counts, normalize with scanly.pp.normalize_total, log-transform, and select the top 2000 highly variable genes via scanpy.pp.highly_variable_genes with flavor='seurat', keeping genes present across all batches. Run each integration method (scDREAMER with ADAM optimizer and batch size 128; baselines with configurations from Supplementary Table 5) on the same preprocessed matrix to produce latent embeddings. Cluster each embedding with Louvain at the resolution maximizing NMI, then compute NMI, ARI, cell-type ASW, batch ASW, PCR batch, graph connectivity, and kBET (k=50 kNN graphs). For scalability, measure runtime in seconds on datasets subsampled to 10k, 100k, 500k, and 1M cells. Aggregate scores by min-max normalizing each metric across methods into composite bio-conservation, batch-correction, and combined composite scores, writing per-method values to a CSV (e.g., integration_metrics_1M_cross_species.csv).

## Related tools

- **scDREAMER** (Primary integration method under benchmark: adversarial variational autoencoder trained with ADAM (batch size 128) producing batch-mixed latent embeddings) — https://github.com/Zafar-Lab/scDREAMER
- **scVI** (Deep generative (VAE) baseline integration method; also a runtime comparison point (~154 s/epoch vs. ~130 s/epoch for scDREAMER))
- **DESC** (Deep-learning baseline integration method run on the same preprocessed data)
- **iMAP** (GAN + autoencoder baseline integration method run on the same preprocessed data)
- **scANVI** (Deep-learning baseline integration method run on the same preprocessed data)
- **scGEN** (Deep-learning baseline integration method run on the same preprocessed data)
- **scDML** (Deep-learning baseline integration method run on the same preprocessed data)
- **INSCT** (Batch-aware triplet-network integration method used as a runtime baseline (~172 s/epoch on the 1M-cell cross-species dataset)) — https://github.com/lkmklsmn/insct
- **Scanpy** (Preprocessing (normalize_total, log-transform, highly_variable_genes flavor='seurat', top 2000 HVGs) and AnnData data container)
- **Louvain clustering** (Clusters each integrated embedding at the resolution maximizing NMI to compute bio-conservation metrics)

## Examples

```
git clone https://github.com/Zafar-Lab/scDREAMER.git && cd scDREAMER/Environments && conda env create -f scDREAMER.yml && conda activate scdreamer
```

## Evaluation signals

- Every min–max normalized composite metric lies in [0, 1]
- Runtime table covers all down-sampled sizes (10k, 100k, 500k, 1M cells) for each method, reproducing scDREAMER's reported runtime advantage (~130 s/epoch vs. ~154 s for scVI and ~172 s for INSCT on the 1M cross-species dataset)
- Louvain clustering resolution is chosen to maximize NMI before computing ARI/ASW metrics
- All methods consume the identical preprocessed matrix (normalize_total + log-transform + top 2000 HVGs, flavor='seurat', genes present across all batches), ensuring fair comparison
- kBET computed with k=50 kNN graphs; where kBET cannot be computed (e.g., healthy heart atlas), the omission is documented rather than silently dropped

## Limitations

- kBET could not be computed for some methods on the healthy heart atlas dataset due to computational limitations in handling large numbers of cells and batches
- scDREAMER, being deep-learning-based, may require fine-tuning of some parameters for best performance
- The model assumes prior knowledge of the number of batches; unsupervised treatment of batch information is unexplored
- Runtime benchmarking in the manuscript compares scDREAMER against scVI and INSCT specifically, not all baselines

## Evidence

- [methods] HVG selection with seurat flavor: [evidence span withheld — anchor status: not_found]
- [methods] ADAM optimizer and batch size: [evidence span withheld — anchor status: not_found]
- [methods] Louvain resolution maximizing NMI: "We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value."
- [results] Runtime benchmark subsampling design: "we measured the runtime of scDREAMER on different datasets subsampled from the atlas-integration dataset and compared the runtime against that of scVI and INSCT"
- [abstract] 1M-cell runtime finding: "we demonstrate that scDREAMER is scalable and can perform atlas-level cross-species (e.g., human and mouse) integration while being faster than other deep-learning-based methods"
- [methods] kBET computational limitation: "for the healthy heart atlas dataset, kBET could not be computed for some of the methods due to computational limitations in handling large number of cells and batches"
- [supplementary] Reproducibility repo: "All analysis and results presented in the manuscript are available at https://github.com/Zafar-Lab/scDREAMER-reproducibility"
- [readme] README scope of use cases: [evidence span withheld — anchor status: not_found]
