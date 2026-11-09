---
name: gene-similarity-scoring
description: Use when two single-cell pseudotime trajectories (e.g., human and mouse
  bone marrow) have been aligned by CAPITAL tree alignment and you need to score which
  marker genes show conserved versus species-biased expression kinetics.
when_to_use_negative:
- The two trajectories have not yet been aligned — genes_similarity_score requires
  a CAPITAL tree alignment first, not raw expression matrices
- The datasets have different global topological structure (article reports alignment
  accuracy degraded in this benchmarking case), making downstream gene-dynamics scores
  unreliable
- The trajectories contain complex graphs such as cycles or disconnected components,
  which CAPITAL in its present form cannot handle since it deals only with trajectory
  trees
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CAPITAL
  role: Provides capital.tl.genes_similarity_score to score gene expression-dynamics
    conservation on aligned pseudotimes, plus the upstream preprocessing, trajectory_tree,
    and tree_alignment functions it depends on
  repo: https://github.com/ykat0/capital
- name: Python 3.8
  role: Runtime environment for CAPITAL, installed via conda with graphviz (conda
    create -n capital python=3.8 graphviz)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:701b3446e3616f972f92f37a5f2ff68d3471d649b2071e1e1c05485f7235b37e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - Python
  - Python 3.8
  - conda
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-similarity-scoring

## Summary

Quantify cross-experiment/cross-species conservation or divergence of gene expression dynamics along CAPITAL-aligned pseudotime using capital.tl.genes_similarity_score, then rank genes from conserved to divergent. This turns a qualitative tree-alignment figure (e.g., human Velten17 vs. mouse Paul15 bone marrow) into a ranked, quantitative gene-level result.

## When to use

Use when two single-cell pseudotime trajectories (e.g., human and mouse bone marrow) have been aligned by CAPITAL tree alignment and you need to score which marker genes show conserved versus species-biased expression kinetics.

## When NOT to use

- The two trajectories have not yet been aligned — genes_similarity_score requires a CAPITAL tree alignment first, not raw expression matrices
- The datasets have different global topological structure (article reports alignment accuracy degraded in this benchmarking case), making downstream gene-dynamics scores unreliable
- The trajectories contain complex graphs such as cycles or disconnected components, which CAPITAL in its present form cannot handle since it deals only with trajectory trees

## Inputs

- Two preprocessed scRNA-seq datasets with CAPITAL trajectory trees already aligned via capital.tl.tree_alignment (e.g., capital.dataset.velten17 human bone marrow and capital.dataset.paul15 mouse myeloid, in AnnData format)
- List of candidate marker genes whose expression dynamics across the aligned pseudotime are to be compared

## Outputs

- Per-gene similarity scores quantifying conservation of expression dynamics between the aligned trajectories
- Ranked list of genes from most conserved to most divergent kinetics
- Optional per-gene aligned pseudotime expression trend plots from capital.pl.gene_expression_trend

## How to apply

First run the CAPITAL pipeline: capital.tl.preprocessing on each dataset, capital.tl.trajectory_tree with capital.tl.dpt pseudotime to build a minimum-spanning-tree trajectory per species, and capital.tl.tree_alignment (dynamic time warping extended to trees) to align them. Then apply capital.tl.genes_similarity_score to candidate marker genes (e.g., erythrocyte, monocyte, neutrophil lineage markers such as IRF7, CSF1, ELANE) using the aligned pseudotimes as input. The score measures similarity of expression dynamics across the alignment; sort genes by score to rank conserved versus divergent kinetics. Interpret high-similarity genes as evolutionarily conserved dynamics and low-similarity genes as species-biased (e.g., IRF7 and CSF1 higher in human, ELANE higher in mouse); visualize per-gene trends with capital.pl.gene_expression_trend. Note that Spearman's correlation is what CAPITAL computes (rather than mutual-nearest-neighbor approaches like Scanorama/Seurat), so the scoring is correlation-based on aligned kinetics.

## Related tools

- **CAPITAL** (Provides capital.tl.genes_similarity_score to score gene expression-dynamics conservation on aligned pseudotimes, plus the upstream preprocessing, trajectory_tree, and tree_alignment functions it depends on) — https://github.com/ykat0/capital
- **Python 3.8** (Runtime environment for CAPITAL, installed via conda with graphviz (conda create -n capital python=3.8 graphviz))

## Examples

```
conda create -n capital python=3.8 graphviz; conda activate capital; pip install capital; # then in Python: score = capital.tl.genes_similarity_score(adata_human_aligned, adata_mouse_aligned, genes=['IRF7','CSF1','ELANE'])
```

## Evaluation signals

- Scores reproduce the paper's reported species-biased kinetics: IRF7 and CSF1 score higher (human-biased) and ELANE scores lower/mouse-biased in the Velten17-vs-Paul15 alignment
- Ranked conserved genes correspond to erythrocyte, monocyte, and neutrophil marker subclasses reported as conserved along the CAPITAL-aligned pseudotime
- Upstream check: rendered tree_alignment.png matches the published trajectory-tree topology with conserved lineage branches aligning between species before any gene scoring is trusted
- Per-gene trend plots (capital.pl.gene_expression_trend) qualitatively agree with the quantitative similarity score direction (high score = similar trends, low score = divergent trends)

## Limitations

- Alignment accuracy of CAPITAL degraded for datasets with different global topological structure in benchmarking, which would propagate into unreliable gene similarity scores
- CAPITAL computes a trajectory as a minimum spanning tree from clusters, and the authors note this part of the algorithm may not be the best method
- CAPITAL in its present form can deal only with trajectory trees, not complex graphs such as cycles or disconnected graphs
- Denoising/imputing expression counts tended to generate more predicted clusters than previously reported, including intermediate states, which can affect trajectory and score stability

## Evidence

- [methods] tl.genes_similarity_score: "Calculate gene similarity
~~~~~~~~~~~~~~~~~~~~~~~~~


.. autosummary::
   :toctree: .

   tl.genes_similarity_score"
- [abstract] species gene dynamics finding: "revealing various gene expression dynamics including gene–gene correlation conservation between different species"
- [discussion] Spearman correlation basis: "Scanorama and Seurat calculate mutual nearest neighbors in a low-dimensional space, whereas CAPITAL computes Spearman’s correlation"
- [readme] README install command: [evidence span withheld — anchor status: external_doc]
- [discussion] topology limitation: "alignment accuracy of CAPITAL for datasets with different global topological structure degraded in our benchmarking test"
