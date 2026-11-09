---
name: embedding-stability-assessment
description: Use when comparing dimensionality-reduction methods on synthetic or single-cell
  datasets and you must verify that cluster shapes, relative positions, and noise
  handling are reproducible across runs rather than artifacts of a single initialization.
when_to_use_negative:
- The dataset already has a trusted ground-truth embedding and the goal is downstream
  classification rather than assessing embedding reproducibility
- The embedding method's output will be judged only on local neighbor preservation
  without concern for global structure or initialization sensitivity
tools:
- name: ivis
  role: Siamese neural-network dimensionality-reduction algorithm fit repeatedly with
    different initializations to test embedding stability; preserves global structure
    on synthetic benchmarks
  repo: https://github.com/beringresearch/ivis
- name: t-SNE
  role: Comparison embedding (scikit-learn, default perplexity) used as a negative
    control expected to produce spurious clusters under random noise
- name: Python
  role: Implementation language for dataset generation, polynomial feature expansion,
    embedding, and scatter-plot rendering
- name: scikit-learn
  role: Provides t-SNE and MinMaxScaler scaling to [0, 1] prior to embedding
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:852529e9225171e5e7aac3199404c22f47b914a5038b02126b5a02513d44b94f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  - scikit-learn
  - t-SNE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# embedding-stability-assessment

## Summary

Assesses whether a dimensionality-reduction embedding (ivis, t-SNE) is stable across random initializations and faithful to global data structure, using repeated fits on synthetic benchmarks (random noise, Cassini, Smiley) expanded with polynomial features.

## When to use

Use when comparing dimensionality-reduction methods on synthetic or single-cell datasets and you must verify that cluster shapes, relative positions, and noise handling are reproducible across runs rather than artifacts of a single initialization.

## When NOT to use

- The dataset already has a trusted ground-truth embedding and the goal is downstream classification rather than assessing embedding reproducibility
- The embedding method's output will be judged only on local neighbor preservation without concern for global structure or initialization sensitivity

## Inputs

- Synthetic 2D benchmark datasets (uniform random noise, Cassini, Smiley) with 5000 points, expanded to nine features via polynomial combinations (degree <= 10)
- High-dimensional single-cell expression matrices (e.g. CyTOF Levine-32-dim, scRNA-seq MNIST/melanoma) scaled to [0, 1] with MinMaxScaler

## Outputs

- Side-by-side 2D scatter-plot PNGs of ivis vs t-SNE embeddings colored by cluster membership or spatial position
- Overlay figure of ten repeated ivis Smiley embeddings demonstrating run-to-run stability (Supplementary Fig. S2 analogue)

## How to apply

Generate base 2D synthetic datasets (uniform random noise, Cassini, Smiley) with 5000 points at a fixed random seed, then expand to nine features using polynomial combinations (degree <= 10) of the original features. Fit ivis with default parameters (as the methods state, 'In all cases default parameters were used to fit multiple models') and t-SNE with default perplexity via scikit-learn, rendering side-by-side 2D scatter plots colored by cluster membership. Repeat the ivis embedding of the Smiley dataset ten times with different random initializations and overlay the embeddings to check that cluster shapes and relative positions are stable. Rationale: the article shows ivis is robust to uniform random noise while t-SNE tends to yield spurious clusters with clean boundaries, so a correct application reproduces global structure in ivis panels and flags spurious clustering in t-SNE panels.

## Related tools

- **ivis** (Siamese neural-network dimensionality-reduction algorithm fit repeatedly with different initializations to test embedding stability; preserves global structure on synthetic benchmarks) — https://github.com/beringresearch/ivis
- **t-SNE** (Comparison embedding (scikit-learn, default perplexity) used as a negative control expected to produce spurious clusters under random noise)
- **Python** (Implementation language for dataset generation, polynomial feature expansion, embedding, and scatter-plot rendering)
- **scikit-learn** (Provides t-SNE and MinMaxScaler scaling to [0, 1] prior to embedding)

## Examples

```
from ivis import Ivis; from sklearn.preprocessing import MinMaxScaler; X = MinMaxScaler().fit_transform(X); emb = Ivis(embedding_dims=2, k=15).fit_transform(X)
```

## Evaluation signals

- ivis panels on the random-noise control show no artificial clusters, while t-SNE panels show spurious compact clusters with clean boundaries
- Cassini and Smiley cluster shapes and relative locations are preserved in the ivis embedding space
- The ten overlaid ivis Smiley embeddings are qualitatively consistent (overlapping cluster positions across seeds)
- All models were fit with default parameters, matching the methods statement
- Each synthetic dataset contains 5000 points expanded to nine features via polynomial combinations (degree <= 10)

## Limitations

- Stability assessment is qualitative (visual overlay of repeated embeddings), not a quantitative metric
- t-SNE spurious-cluster behavior is a known feature of algorithms preserving pairwise (dis)similarities and may generalize to methods like LargeVis
- Hyperparameter sensitivity must be checked separately; the article notes robustness to hyperparameter settings is shown in supplementary figures

## Evidence

- [discussion] spurious clusters with random noise: "in the presence of random noise, the algorithm tends to yield spurious clusters with clean boundaries, potentially hindering accurate interpretation and discovery"
- [discussion] Smiley global structure preserved: "in the synthetic Smiley dataset, ivis preserved both the shape and relative locations of each cluster in the embedding space"
- [methods] polynomial feature expansion: "we will artificially add reduntant features to the original datasets using polynomial combinations (degree ≤ 10) of the original features."
- [methods] default parameters: "In all cases default parameters were used to fit multiple models"
- [readme] ivis README invocation: "model = Ivis(embedding_dims=2, k=15)

embeddings = model.fit_transform(X_scaled)"
