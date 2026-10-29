---
name: tsne-spurious-cluster-contrast
description: Use when evaluating whether a dimensionality-reduction method artificially
  fragments homogeneous data into compact clusters, or when validating that global
  structure (cluster shapes and relative locations) survives 2D embedding of high-dimensional
  data.
when_to_use_negative:
- Input is real single-cell expression data (e.g. CyTOF or scRNA-seq) requiring biological
  interpretation rather than a methodological cluster-artifact contrast
- The question concerns quantitative distance preservation metrics (e.g. Mantel test
  PCC or SVM classification accuracy) rather than visual demonstration of spurious
  clustering
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: ivis
  role: Produces the reference 2D embedding that preserves global structure and cluster
    shapes/relative locations
  repo: https://github.com/beringresearch/ivis
- name: t-SNE
  role: Comparison embedding (scikit-learn, default perplexity) expected to yield
    spurious clusters with clean boundaries under random noise
- name: scikit-learn
  role: Provides the t-SNE implementation and MinMaxScaler preprocessing
- name: Python
  role: Language for generating synthetic datasets (NumPy), feature expansion, embedding,
    and matplotlib plotting
  repo: https://github.com/beringresearch/ivis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:afa19832730ad33d79f434d01bdbcdee701fb4fdc38ebde04a226018cf4c5f8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  - scikit-learn
  - t-SNE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# tsne-spurious-cluster-contrast

## Summary

A benchmark-style visualisation skill that contrasts t-SNE and ivis embeddings on synthetic datasets to expose t-SNE's tendency to create spurious clusters under uniform random noise while ivis preserves global structure.

## When to use

Use when evaluating whether a dimensionality-reduction method artificially fragments homogeneous data into compact clusters, or when validating that global structure (cluster shapes and relative locations) survives 2D embedding of high-dimensional data.

## When NOT to use

- Input is real single-cell expression data (e.g. CyTOF or scRNA-seq) requiring biological interpretation rather than a methodological cluster-artifact contrast
- The question concerns quantitative distance preservation metrics (e.g. Mantel test PCC or SVM classification accuracy) rather than visual demonstration of spurious clustering

## Inputs

- Synthetic 2D datasets (uniform random noise, Cassini, Smiley), 5000 points each, generated in Python/NumPy with a fixed random seed
- Nine-dimensional expanded datasets built by polynomial feature expansion (degree ≤ 10) of the base 2D coordinates

## Outputs

- Side-by-side 2D scatter plots (PNG) of ivis vs t-SNE embeddings for each dataset, colored by cluster membership or spatial position
- Overlaid ten-run ivis embeddings of the Smiley dataset demonstrating cross-initialization stability (Supplementary Fig. S2 analogue)

## How to apply

Generate the three base two-dimensional synthetic datasets — uniform random noise (control), Cassini, and Smiley — with 5000 points each in Python/NumPy using a fixed random seed. Expand each to nine features by adding redundant features via polynomial combinations (degree ≤ 10) of the original features. Embed each nine-dimensional dataset in two dimensions with ivis and with t-SNE (scikit-learn, default perplexity); the paper notes 'In all cases default parameters were used to fit multiple models'. Scale inputs to [0, 1] with MinMaxScaler before ivis, as shown in the README workflow. Render side-by-side 2D scatter plots colored by cluster membership or spatial position. To assess stability, repeat the ivis embedding of the Smiley dataset ten times with different random initializations and overlay the results, reproducing the Supplementary Fig. S2 analysis.

## Related tools

- **ivis** (Produces the reference 2D embedding that preserves global structure and cluster shapes/relative locations) — https://github.com/beringresearch/ivis
- **t-SNE** (Comparison embedding (scikit-learn, default perplexity) expected to yield spurious clusters with clean boundaries under random noise)
- **scikit-learn** (Provides the t-SNE implementation and MinMaxScaler preprocessing)
- **Python** (Language for generating synthetic datasets (NumPy), feature expansion, embedding, and matplotlib plotting) — https://github.com/beringresearch/ivis

## Examples

```
from ivis import Ivis; from sklearn.preprocessing import MinMaxScaler; X_scaled = MinMaxScaler().fit_transform(X); model = Ivis(embedding_dims=2, k=15); embeddings = model.fit_transform(X_scaled)
```

## Evaluation signals

- The ivis panel of the uniform random-noise control shows no artificial clusters, while the t-SNE panel shows spurious clusters with clean boundaries
- ivis panels of the Cassini and Smiley datasets retain cluster shapes and relative cluster locations from the original 2D data
- The ten repeated ivis embeddings of the Smiley dataset are qualitatively consistent (overlaid plots show stable shapes and positions)
- All datasets are 9-dimensional before embedding (3 base features expanded with polynomial combinations, degree ≤ 10) and embedded to exactly 2 dimensions
- Figures are saved as PNG side-by-side scatter plots colored by cluster membership or spatial position

## Limitations

- t-SNE's spurious clusters are demonstrated specifically in the presence of uniform random noise; clean structured data may not show the artifact
- Similar spurious clustering may be a general feature of algorithms preserving pairwise (dis)similarities (e.g. LargeVis), not t-SNE alone
- The demonstration relies on default hyperparameters and qualitative visual inspection of scatter plots rather than quantitative metrics

## Evidence

- [discussion] t-SNE spurious clusters under noise: "in the presence of random noise, the algorithm tends to yield spurious clusters with clean boundaries, potentially hindering accurate interpretation and discovery (Fig. 1C)"
- [discussion] ivis preserves Smiley structure: "For example, in the synthetic Smiley dataset, ivis preserved both the shape and relative locations of each cluster in the embedding space"
- [methods] Polynomial feature expansion: "we will artificially add reduntant features to the original datasets using polynomial combinations (degree ≤ 10) of the original features."
- [readme] README ivis vs t-SNE global structure: [evidence span withheld — anchor status: external_doc]
