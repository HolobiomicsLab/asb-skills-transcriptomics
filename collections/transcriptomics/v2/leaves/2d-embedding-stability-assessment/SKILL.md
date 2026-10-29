---
name: 2d-embedding-stability-assessment
description: Decision support for judging whether a GraphDDP 2D embedding depends
  critically on the class-confidence bias - required before trusting layouts built
  from prior cluster labels.
when_to_use_negative:
- No prior class labels exist for the instances - the confidence bias contracts only
  same-label edges, so an unlabeled dataset has nothing to sweep
- Only a single embedding at one parameter setting is needed - a full sweep is unnecessary;
  just run graph_embed once at the default -c 1.0
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0091
tools:
- name: GraphEmbed
  role: Executes the confidence-parameter sweep; each run with a different -c value
    produces one 2D layout whose stability is assessed
  repo: github.com/fabriziocosta/GraphEmbed
- name: graph_embed
  role: Python2/conda (bioconda) variant of the same embedding tool used to generate
    layouts at each C
  repo: http://conda.pydata.org/miniconda.html
- name: pygraphviz
  role: Non-pip dependency required for the graph-layout rendering used in the embedding
    images
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05988-7
    title: 'GraphDDP: a graph-embedding approach to detect differentiation pathways
      in single-cell-data using prior class knowledge'
  source_content_hash: sha256:59c5145107707e34e88de03907c1ebeea2145077dd095f17141388b3c7f38db0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphDDP
  - GraphEmbed
  - graph_embed
  - pygraphviz
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# 2D embedding stability assessment

## Summary

Sweeps the GraphDDP/GraphEmbed class-confidence parameter (-c) across its recommended regimes and verifies that the Kamada-Kawai 2D layout is robust, with same-class instances contracting as C increases (d/(1+C)).

## When to use

Decision support for judging whether a GraphDDP 2D embedding depends critically on the class-confidence bias - required before trusting layouts built from prior cluster labels.

## When NOT to use

- No prior class labels exist for the instances - the confidence bias contracts only same-label edges, so an unlabeled dataset has nothing to sweep
- Only a single embedding at one parameter setting is needed - a full sweep is unnecessary; just run graph_embed once at the default -c 1.0

## Inputs

- Input data matrix CSV (e.g. example/prot_expression.csv)
- Class/target label CSV (e.g. example/target.csv)
- Sweep of class-confidence values C (0.5-1 for clean data; 5-30 for noisy data)

## Outputs

- Per-C timestamped output directories containing 2D_coords.txt (one line per instance)
- Per-C target.txt class identifier files
- Per-C img_1_clean.pdf 2D embedding images for qualitative compaction/stability comparison

## How to apply

Run graph_embed repeatedly over a sweep of the class-confidence parameter, covering C=0.5-1 for clean, well-separated data and C=5-30 for noisy data where a strong separation must be forced in the 2D representation. Hold all other settings fixed (e.g. --random_state=1, -k=3 same-class links, -d=1 denser different-class links within a -z=5 horizon, -l=3 mutual-neighbor outlier detection) so that only the confidence bias varies. Internally, the method contracts the desired same-class edge length to d/(1+C) before running the Kamada-Kawai spring-energy minimization on the Dijkstra-computed graph distances. For each C, inspect the timestamped -o output directory: higher C should yield visibly more compact same-class hulls/regions, and the overall arrangement of classes and trajectories should remain stable across the whole range of C values (as documented in Supplementary Fig. 1). Confirm that all per-C output files (2D_coords.txt, target.txt, img_1_clean.pdf) were produced before declaring the layout robust.

## Related tools

- **GraphEmbed** (Executes the confidence-parameter sweep; each run with a different -c value produces one 2D layout whose stability is assessed) — github.com/fabriziocosta/GraphEmbed
- **graph_embed** (Python2/conda (bioconda) variant of the same embedding tool used to generate layouts at each C) — http://conda.pydata.org/miniconda.html
- **pygraphviz** (Non-pip dependency required for the graph-layout rendering used in the embedding images)

## Examples

```
./graph_embed -i example/prot_expression.csv -t example/target.csv -c 1.5 -k 5 -z 10 -l 0 --random_state 2 --normalization --correlation_transformation --feature_selection
```

## Evaluation signals

- Higher C values yield visibly more compact same-class layouts, matching the documented behavior that values higher than 0 indicate stronger belief in the supervised information
- The qualitative arrangement of classes and differentiation trajectories remains stable across the swept range of C (Supplementary Fig. 1 behavior)
- For every swept C, the timestamped output directory contains 2D_coords.txt, target.txt, and img_1_clean.pdf
- 2D_coords.txt has exactly one coordinate line per input instance and target.txt one target line per instance
- All sweep runs use identical preprocessing and --random_state so differences are attributable to C alone

## Limitations

- Robustness is qualitative (visual compaction/stability), not backed by quantitative metrics - the paper reports no numeric scores comparing layouts
- Very high C can be needed only for noisy data; forcing strong separation on naturally well-separated clusters may over-contract the layout
- Edge construction with -l=0 disables outlier detection, so outlier-driven layout artifacts may confound stability judgments

## Evidence

- [readme] README confidence semantics: "Values higher than 0 indicate a stronger belief and will result in more compact layouts for instances of the same class."
- [readme] Recommended C regimes: "Values of 0.5-1 are suitable for clean data where clusters are naturally well separated, values of 5-30 are suitable for noisy data"
- [readme] Same-class edge contraction rule: "If the endpoints of an edge have the same label then the desired distance is divided by 1 + class_confidence."
- [methods] Confidence CLI parameter: "To set a desired separation strength use -c, to specify the number of neighbors use -k"
- [methods] Timestamped output directories: "The option -o allows to set a directory where the produced files are placed. The directory name is suffixed with a time stamp"
