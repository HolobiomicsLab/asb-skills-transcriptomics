---
name: cluster-class-assignment-preprocessing
description: Use when a single-cell expression matrix from a prior clustering step
  must be paired with per-cell class labels and preprocessed (normalization, correlation
  transformation, feature selection) before a class-informed 2D embedding.
when_to_use_negative:
- When no prior cluster assignment exists — GraphDDP starts from a user-defined cluster
  assignment and cannot be run without the target CSV
- When the goal is an unsupervised visualization (plain t-SNE, PCA, MDS) that ignores
  class information rather than a class-informed embedding
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GraphDDP
  role: Method that consumes the preprocessed expression matrix plus class labels
    to build knn and k_shift edges for the supervised 2D layout
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: GraphEmbed
  role: Installable implementation (PyPI pip install graphembed; conda graph_embed
    -c bioconda) of graph_embed.py that performs normalization, correlation transformation,
    and feature selection
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: pygraphviz
  role: Non-pip dependency required for graph layout output (conda install pygraphviz)
- name: Monocle2
  role: Comparison pseudotime method rerun on the same expression data (min_expr=20,
    DDRTree) to contrast against the class-informed embedding
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:649e4d93dd234efcf44e50e6d55f61bb9d14239f035d827e3102dc60a81ad2c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabriziocosta/GraphEmbed
  tools:
  - GraphDDP
  - GraphEmbed
  - Monocle2
  - pygraphviz
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cluster-class-assignment-preprocessing

## Summary

Prepare prior class knowledge (cluster assignments from RaceID3 or similar clustering) and the single-cell expression matrix as CSV inputs for the GraphDDP/GraphEmbed supervised graph-embedding, applying normalization, correlation transformation, and discriminative feature selection.

## When to use

Use when a single-cell expression matrix from a prior clustering step must be paired with per-cell class labels and preprocessed (normalization, correlation transformation, feature selection) before a class-informed 2D embedding.

## When NOT to use

- When no prior cluster assignment exists — GraphDDP starts from a user-defined cluster assignment and cannot be run without the target CSV
- When the goal is an unsupervised visualization (plain t-SNE, PCA, MDS) that ignores class information rather than a class-informed embedding

## Inputs

- Single-cell expression matrix in CSV format (e.g. Velten mouse hematopoietic progenitor data)
- Class/target CSV file with one discrete cluster label per instance (e.g. RaceID3 cluster assignments)

## Outputs

- Normalized and correlation-transformed feature matrix restricted to discriminative selected features
- Class-filtered instance set satisfying min_threshold=5 and max_threshold=300 instances per class
- Input pair (-i expression.csv, -t target.csv) ready for graph_embed.py

## How to apply

Export the expression matrix (e.g. the Velten mouse hematopoietic progenitor matrix) as a CSV and the cluster assignments (e.g. RaceID3-derived labels such as HSC, CLP, GMP) as a separate class/target CSV consumed via -i and -t by graph_embed.py. Apply the data transformations with the --normalization, --correlation_transformation, and --feature_selection flags; the correlation transformation converts the matrix to a correlation-coefficient matrix, and density for edge construction is defined as the sum of pairwise cosine similarities between an instance and all other instances. Bound class sizes with --min_threshold=5 and --max_threshold=300 to drop or cap classes with too few or too many instances, and fix --random_state for reproducibility. Tune the class separation strength -c (0.5-1 for well-separated clusters, 5-30 for noisy data), the same-class neighbor links -k (default 3), the denser-neighbor links -d, the density-gradient search horizon -z (default 5), and the outlier mutual-neighbor count -l (default 3).

## Related tools

- **GraphDDP** (Method that consumes the preprocessed expression matrix plus class labels to build knn and k_shift edges for the supervised 2D layout) — https://github.com/fabriziocosta/GraphEmbed
- **GraphEmbed** (Installable implementation (PyPI pip install graphembed; conda graph_embed -c bioconda) of graph_embed.py that performs normalization, correlation transformation, and feature selection) — https://github.com/fabriziocosta/GraphEmbed
- **pygraphviz** (Non-pip dependency required for graph layout output (conda install pygraphviz))
- **Monocle2** (Comparison pseudotime method rerun on the same expression data (min_expr=20, DDRTree) to contrast against the class-informed embedding)

## Examples

```
./graph_embed.py -i expression.csv -t target.csv -c 1 -k 3 -z 5 -l 3 --random_state 2 --normalization --correlation_transformation --feature_selection --min_threshold=5 --max_threshold=300
```

## Evaluation signals

- The target CSV contains exactly one discrete label per instance matching the rows of the expression CSV
- After filtering, every retained class has between min_threshold=5 and max_threshold=300 instances
- A timestamped output directory is produced containing 2D_coords.txt, target.txt, and img_1_clean.pdf through img_5_hull_link.pdf plus a log file
- Setting a fixed --random_state reproduces the same 2D embedding on rerun
- On the mouse HSPC benchmark, the embedding with class convex hulls shows the HSC-centered branching hierarchy with megakaryocyte, erythrocyte, LMPP/B-cell and GMP branches

## Limitations

- No quantitative performance metric is reported for the resulting embedding; validation against the paper benchmark is visual/qualitative (convex-hull images)
- No dataset accessions or software versions for compared tools (TSCAN, Monocle2, SPRING) are given, complicating exact reproduction
- Class confidence and neighbor parameters must be chosen empirically: 0.5-1 suits well-separated clusters, 5-30 noisy data; mis-set values distort the layout

## Evidence

- [readme] Input flags and transformations: "./graph_embed.py -i example/prot_expression.csv -t example/target.csv"
- [readme] Class size bounds: "--min_threshold=N                 Min num instances per class [default: 5]"
- [readme] Separation strength guidance: "Values of 0.5-1 are suitable for clean data where clusters are naturally well separated, values of 5-30 are suitable for noisy data"
- [readme] Density definition: "Density is defined as the sum of the pairwise cosine similarity between an instance and all the other instances."
- [abstract] Prior class knowledge requirement: "GraphDDP starts from a user-defined cluster assignment and then uses a force-based graph layout approach"
