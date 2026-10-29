---
name: hematopoietic-progenitor-hierarchy-interpretation
description: Use when scRNA-seq data of hematopoietic progenitors (e.g. the Velten
  multipotent progenitor dataset with RaceID3 cluster assignments) needs a lineage-hierarchy
  readout, or when benchmarking pseudotime tools like Monocle2 against a class-informed
  graph embedding.
when_to_use_negative:
- Input is a single continuous trajectory without prior class/cluster labels — GraphDDP
  requires a discrete label for each instance as user-defined cluster assignment
- The goal is a quantitative pseudotime benchmark — the article reports no quantitative
  metric for the compared tools, so this skill only yields visual/qualitative judgments
- Data are well outside hematopoietic lineages where the HSC-centered reference hierarchy
  (HSC, CLP, GMP, megakaryocyte, erythrocyte, LMPP/B-cell) cannot serve as the ground
  truth for interpretation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: GraphDDP
  role: Class-informed graph-embedding (knn + k_shift edges, force-based Kamada-Kawai
    layout) that yields the HSC-centered lineage visualization on RaceID3 clusters
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: GraphEmbed
  role: Installable implementation (PyPI pip install graphembed; python2 variant via
    conda graph_embed -c bioconda) providing graph_embed.py
  repo: https://github.com/fabriziocosta/GraphEmbed
- name: pygraphviz
  role: Non-pip dependency required for graph rendering during installation
- name: Monocle2
  role: Pseudotime/DDRTree comparison method rerun with min_expr=20 and marker-based
    class assignment to expose its CLP-separate-branch and GMP-outgroup artifacts
- name: RaceID3
  role: Upstream clustering whose cluster assignments are consumed as the target/classes
    file for GraphDDP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:952b36381bc626421b76f65f8010b176bfa95e13b4f4610a5345651aab0cf21f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabriziocosta/GraphEmbed
  tools:
  - GraphDDP
  - GraphEmbed
  - Monocle2
  - RaceID3
  - pygraphviz
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hematopoietic-progenitor-hierarchy-interpretation

## Summary

Interpret a recovered single-cell lineage embedding of hematopoietic progenitors against the known mouse HSC-centered hierarchy (megakaryocyte, erythrocyte, LMPP/B-cell and GMP branches), using GraphDDP embeddings of RaceID3 clusters and a Monocle2 DDRTree rerun as the comparison layout.

## When to use

Use when scRNA-seq data of hematopoietic progenitors (e.g. the Velten multipotent progenitor dataset with RaceID3 cluster assignments) needs a lineage-hierarchy readout, or when benchmarking pseudotime tools like Monocle2 against a class-informed graph embedding.

## When NOT to use

- Input is a single continuous trajectory without prior class/cluster labels — GraphDDP requires a discrete label for each instance as user-defined cluster assignment
- The goal is a quantitative pseudotime benchmark — the article reports no quantitative metric for the compared tools, so this skill only yields visual/qualitative judgments
- Data are well outside hematopoietic lineages where the HSC-centered reference hierarchy (HSC, CLP, GMP, megakaryocyte, erythrocyte, LMPP/B-cell) cannot serve as the ground truth for interpretation

## Inputs

- Velten mouse hematopoietic/multipotent progenitor expression matrix as CSV (-i <expression.csv>)
- RaceID3-derived cluster assignments per cell as target CSV (-t <target.csv>)
- Marker-gene-based class labels (HSC, CLP, GMP, megakaryocyte, erythrocyte, LMPP/B-cell) for the Monocle2 rerun

## Outputs

- Timestamped GraphDDP output directory with 2D_coords.txt, target.txt, log and img_1_clean.pdf through img_5_hull_link.pdf (knn edges, shift-edges, class convex hulls)
- Monocle2 DDRTree pseudotime layout of the same expression data colored by marker-assigned classes
- Qualitative comparison verdict: GraphDDP HSC-centered hierarchy vs Monocle2 CLP-separate-branch / GMP-outgroup discrepancy

## How to apply

Run GraphDDP on the Velten mouse progenitor expression matrix (CSV) plus RaceID3 cluster assignments via graph_embed.py, using -c 0.5-1 for the well-separated progenitor classes (5-30 only for noisy data), -k 3, -d for denser-neighbor links, -z 5, and -l 3, with --normalization, --correlation_transformation and --feature_selection, --min_threshold=5, --max_threshold=300 and a fixed --random_state. Inspect the timestamped output directory's 2D_coords.txt, target.txt and the img_1_clean through img_5_hull_link PDFs, focusing on knn edges, shift-edges and class convex hulls to read the HSC-centered branching architecture. Rerun Monocle2 on the same expression matrix with min_expr=20 and the DDRTree layout, assigning classes by marker genes, to obtain the comparison pseudotime tree. Judge correctness qualitatively: GraphDDP should place HSCs at the center with megakaryocyte, erythrocyte, LMPP/B-cell and GMP branches radiating outward, while Monocle2 is expected to separate CLPs from B cells on distinct branches and push GMPs out as an outgroup — the reported discrepancy that motivates the comparison. No quantitative metric is reported, so both checks are visual and hinge on convex-hull adjacency of lineages in the GraphDDP embedding.

## Related tools

- **GraphDDP** (Class-informed graph-embedding (knn + k_shift edges, force-based Kamada-Kawai layout) that yields the HSC-centered lineage visualization on RaceID3 clusters) — https://github.com/fabriziocosta/GraphEmbed
- **GraphEmbed** (Installable implementation (PyPI pip install graphembed; python2 variant via conda graph_embed -c bioconda) providing graph_embed.py) — https://github.com/fabriziocosta/GraphEmbed
- **pygraphviz** (Non-pip dependency required for graph rendering during installation)
- **Monocle2** (Pseudotime/DDRTree comparison method rerun with min_expr=20 and marker-based class assignment to expose its CLP-separate-branch and GMP-outgroup artifacts)
- **RaceID3** (Upstream clustering whose cluster assignments are consumed as the target/classes file for GraphDDP)

## Examples

```
./graph_embed.py -i velten_expression.csv -t raceid3_targets.csv -c 1 -k 3 -d 1 -z 5 -l 3 --normalization --correlation_transformation --feature_selection --min_threshold=5 --max_threshold=300 --random_state 2
```

## Evaluation signals

- GraphDDP output directory contains 2D_coords.txt, target.txt, log and all five images img_1_clean.pdf through img_5_hull_link.pdf
- img_2/img_3 edge images show red knn edges and blue k-shift edges connecting the expected lineage pairs; k-shift edges to outliers (instances without mutual k neighbors, -l 3) are deleted
- img_4_hull/img_5_hull_link convex hulls show HSCs centrally placed with megakaryocyte, erythrocyte, LMPP/B-cell and GMP hulls branching outward, reproducing the known hierarchy
- Monocle2 DDRTree rerun (min_expr=20) qualitatively reproduces the reported discrepancy: CLPs on a branch separate from B cells and GMPs as an outgroup
- Class sizes pass the --min_threshold=5 / --max_threshold=300 bounds and the run is reproducible under the fixed --random_state

## Limitations

- Validation is purely visual/qualitative — no quantitative metric is reported in the paper for either the GraphDDP hierarchy or the Monocle2 comparison
- No software versions or parameter settings for the compared tools (TSCAN, Monocle2, SPRING) are given, limiting exact reproducibility of the benchmark
- No dataset accessions are cited for the progenitor data, so the exact input matrices may be hard to obtain; results also depend on the upstream RaceID3 clustering quality
- GraphDDP's -c class-confidence must be tuned: 0.5-1 only suits naturally well-separated clusters, while noisy data needs 5-30 to force separation

## Evidence

- [readme] A knn edge is an edge to the k-th nearest instance that has the same label. A k_shift edge is an edge to the k-th nearest instance that is denser and has a different label.: "A knn edge is an edge to the k-th nearest instance that has the same label. A k_shift edge is an edge to the k-th nearest instance that is denser"
- [readme] The following files are produced: 2D_coords.txt ... img_5_hull_link.pdf The image of the 2D embedding with class convex hulls and first shift-edges.: "img_5_hull_link.pdf The image of the 2D embedding with class convex hulls and first shift-edges."
- [readme] Values of 0.5-1 are suitable for clean data where clusters are naturally well separated, values of 5-30 are suitable for noisy data.: "Values of 0.5-1 are suitable for clean data where clusters are naturally well separated, values of 5-30 are suitable"
- [abstract] GraphDDP integrates both viewpoints in an intuitive visualization, using a force-based graph layout on edges emphasizing cluster membership and differentiation based on density gradients.: "uses a force-based graph layout approach on two types of carefully constructed edges: one emphasizing cluster membership, the other, based on density gradients"
- [methods] Min num instances per class [default: 5]; Max num instances per class [default: 300].: "--min_threshold=N                 Min num instances per class [default: 5]"
