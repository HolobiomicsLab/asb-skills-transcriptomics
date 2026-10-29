---
name: zinb-decoder-autoencoder-training
description: Use when benchmarking decoder choices for scRNA-seq clustering, e.g.
  reproducing the scMGCA ablation that swaps the multinomial decoder for a ZINB likelihood
  on count matrices.
when_to_use_negative:
- When the goal is the full scMGCA baseline — that uses the multinomial-based decoder,
  not ZINB
- When the input is normalized/log-transformed expression rather than raw counts,
  since the ZINB likelihood models count structure and dropout
- When the aim is denoising alone rather than joint representation learning and clustering
  (plain DCA suffices)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scMGCA
  role: Graph convolutional autoencoder framework (two GCN layers, cell-PPMI graph,
    KL-divergence self-optimizing clustering) whose decoder is swapped for the ZINB
    ablation
  repo: https://pypi.org/project/scMGCA
- name: SCANPY
  role: 'Preprocessing of count matrices: filtering unexpressed genes, normalization,
    and selection of top d highly variable genes'
- name: DCA
  role: Reference ZINB deep count autoencoder whose likelihood formulation the ablation
    decoder follows
  repo: https://github.com/theislab/dca
- name: scDeepCluster
  role: Reference model-based deep embedding clustering with ZINB autoencoder that
    the ablation decoder imitates
  repo: https://github.com/ttgump/scDeepCluster
- name: scikit-learn
  role: Provides K-means (with spectral clustering) initialization of cluster centers
    and clustering metrics
- name: TensorFlow
  role: Framework on which the scMGCA core model is built and trained
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:f50dc728e106054789e39fa508ecd794975ece4631ad18c3c2b08c888ac47aec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DCA
  - SCANPY
  - TensorFlow
  - scDeepCluster
  - scMGCA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# zinb-decoder-autoencoder-training

## Summary

Trains a single-cell graph autoencoder whose decoder models count data with a zero-inflated negative binomial (ZINB) likelihood, in the style of DCA and scDeepCluster. In the scMGCA ablation it serves as the decoder-ablation variant against which the multinomial-based decoder is judged.

## When to use

Use when benchmarking decoder choices for scRNA-seq clustering, e.g. reproducing the scMGCA ablation that swaps the multinomial decoder for a ZINB likelihood on count matrices.

## When NOT to use

- When the goal is the full scMGCA baseline — that uses the multinomial-based decoder, not ZINB
- When the input is normalized/log-transformed expression rather than raw counts, since the ZINB likelihood models count structure and dropout
- When the aim is denoising alone rather than joint representation learning and clustering (plain DCA suffices)

## Inputs

- Benchmark scRNA-seq raw count matrices (public repositories)
- Cell-PPMI adjacency graph built by random surfing on the KNN cell graph
- True cell-type label annotations for ARI/NMI evaluation

## Outputs

- Trained ZINB-decoder graph autoencoder model
- Predicted cluster assignments per cell
- ARI and NMI clustering metrics for the decoder-ablation variant
- Rows for ablation_results.csv comparing ZINB vs multinomial decoder

## How to apply

Preprocess benchmark scRNA-seq count matrices in SCANPY (filter genes not expressed in cells, normalize the expression matrix, retain the top d highly variable genes). Build the cell-PPMI graph via random surfing on the KNN graph as in full scMGCA, but replace the multinomial-based decoder with a zero-inflated negative binomial likelihood following the DCA/scDeepCluster formulation, which models count structure, overdispersion and dropout. Train the two-layer GCN autoencoder while jointly performing self-optimized clustering via KL divergence on the latent representation, initializing cluster centers with the best strategy from K-means and spectral clustering; split datasets with >25,000 cells into multiple learning batches. Evaluate by comparing predicted labels to true cell-type labels with ARI and NMI, including on datasets with rare cell types and small clusters ('QS heart', 'Muraro', 'Qx limb Muscle', 'Adam'). The variant is judged correctly reproduced if it underperforms full scMGCA on these metrics, consistent with the paper's ablation comparison in the Source Data file.

## Related tools

- **scMGCA** (Graph convolutional autoencoder framework (two GCN layers, cell-PPMI graph, KL-divergence self-optimizing clustering) whose decoder is swapped for the ZINB ablation) — https://pypi.org/project/scMGCA
- **SCANPY** (Preprocessing of count matrices: filtering unexpressed genes, normalization, and selection of top d highly variable genes)
- **DCA** (Reference ZINB deep count autoencoder whose likelihood formulation the ablation decoder follows) — https://github.com/theislab/dca
- **scDeepCluster** (Reference model-based deep embedding clustering with ZINB autoencoder that the ablation decoder imitates) — https://github.com/ttgump/scDeepCluster
- **scikit-learn** (Provides K-means (with spectral clustering) initialization of cluster centers and clustering metrics)
- **TensorFlow** (Framework on which the scMGCA core model is built and trained)

## Examples

```
pip install scMGCA
```

## Evaluation signals

- The ZINB-decoder variant must underperform full scMGCA on ARI and NMI across most benchmark datasets, matching the paper's ablation table
- Performance degradation is reproduced on datasets with rare cell types and small clusters ('QS heart', 'Muraro', 'Qx limb Muscle', 'Adam')
- The decoder outputs a ZINB likelihood (mean, dispersion, dropout probability) rather than multinomial probabilities
- Cluster centers are initialized with the best strategy from K-means and spectral clustering, as in the original setup
- Compiled metrics can be diffed against the full scMGCA baseline rows from the paper's Source Data file

## Limitations

- Software version numbers (Python, TensorFlow, SCANPY) are not reported in the article, hindering exact reproduction
- Loss weight coefficients γ1, γ2, γ3 and the maximum iteration count for the stopping condition are not specified numerically
- Number of KNN neighbors k for graph construction and the random-surfing restart probability (1 − α) are not given
- Benchmark data are only described as from public repositories with no accessions; the Source Data file is referenced without a DOI or URL

## Evidence

- [intro] DCA ZINB autoencoder formulation: "deep count autoencoder network (DCA) uses an autoencoder based on negative binomial loss, with or without zero expansion to denoise scRNA-seq count data"
- [intro] scDeepCluster joint ZINB clustering: "scDeepCluster learns the feature representation and clustering simultaneously in the autoencoder from the zero-inflated negative binomial distribution model"
- [results] KL divergence self-optimizing clustering: "we perform the Kullback-Leibler (KL) divergence on the latent representations to self-optimize the clustering task"
- [results] Datasets with rare cell types: [evidence span withheld — anchor status: not_found]
- [readme] DCA README on ZINB loss: [evidence span withheld — anchor status: external_doc]
