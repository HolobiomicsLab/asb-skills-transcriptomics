---
name: cell-type-marker-enrichment-analysis
description: Use when you have a cell-type-specific gene network (weighted edge matrix)
  for one of 26 cell types and need to verify that its node scores recover the correct
  pangaloDB cell-type markers.
when_to_use_negative:
- When the goal is to test whether TFs are hubs or whether TF targets are enriched
  (use ENCODE-target and TF-enrichment preranks instead)
- When the network is not cell-type-specific (e.g., a single atlas-level literature-based
  network), since marker enrichment per cell type is not defined
- When you need a quantitative significance judgment per cell type rather than the
  paper's criterion of manual identification of the cell type name in the top 10
edam_operation: http://edamontology.org/operation_3932
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scPRINT
  role: source of the benchmarked cell-type-specific inferred networks whose nodes
    are tested for marker enrichment
  repo: https://github.com/cantinilab/scPRINT
- name: BenGRN
  role: provides the prerank setup for deriving per-gene ordered scores from inferred
    gene networks
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: benchmarking suite used with BenGRN to score and evaluate the inferred gene
    networks
  repo: https://github.com/cantinilab/GRnnData
- name: Python
  role: implementation environment for prerank scoring, GSEA, and aggregation of the
    26-cell-type count tables
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:9050083d0bebc357185efbee0df453900512991a82bcd579b3925bcffec8ab66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - BenGRN
  - GRnnData
  - Python
  - scPRINT
  edam_operation: http://edamontology.org/operation_3932
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cell-type-marker-enrichment-analysis

## Summary

Tests whether a cell-type-specific gene network inferred by a method (scPRINT, scGPT, Geneformer, GENIE3, DeepSEM, etc.) is enriched for that cell type's marker genes, using GSEA prerank on network node scores against pangaloDB marker gene sets, aggregated across the 26 cell types benchmarked in the scPRINT paper.

## When to use

Use when you have a cell-type-specific gene network (weighted edge matrix) for one of 26 cell types and need to verify that its node scores recover the correct pangaloDB cell-type markers.

## When NOT to use

- When the goal is to test whether TFs are hubs or whether TF targets are enriched (use ENCODE-target and TF-enrichment preranks instead)
- When the network is not cell-type-specific (e.g., a single atlas-level literature-based network), since marker enrichment per cell type is not defined
- When you need a quantitative significance judgment per cell type rather than the paper's criterion of manual identification of the cell type name in the top 10

## Inputs

- cell-type-specific weighted gene network (edge-score matrix) per method, per cell type (26 cell types)
- pangaloDB cell-type marker gene sets
- all genes in the network set as the prerank background

## Outputs

- per-cell-type GSEA prerank enrichment results against pangaloDB markers
- aggregated enrichment count table across methods and all 26 cell types (correct cell type ranked in top 10: yes/no)

## How to apply

For each method's cell-type-specific network, derive per-gene ordered scores using the prerank setup of BenGRN/GRnnData: sum edge scores row-wise (Target-Hub) or column-wise (Regulator-Hub), or compute eigenvector centrality with NetworkX, using all genes in the network as the prerank background. Run GSEA prerank on these node scores against pangaloDB cell-type marker gene sets for that cell type. Repeat for all 26 cell types and aggregate results into a count table, judging a network cell-type-enriched when its cell type name is manually identified among the top 10 enriched cell types in the enrichment results. Compare enrichment counts across methods (e.g., scPRINT and scGPT vs. GENIE3, DeepSEM, Geneformer v2) to assess whether inferred networks are biologically meaningful.

## Related tools

- **scPRINT** (source of the benchmarked cell-type-specific inferred networks whose nodes are tested for marker enrichment) — https://github.com/cantinilab/scPRINT
- **BenGRN** (provides the prerank setup for deriving per-gene ordered scores from inferred gene networks) — https://github.com/jkobject/benGRN
- **GRnnData** (benchmarking suite used with BenGRN to score and evaluate the inferred gene networks) — https://github.com/cantinilab/GRnnData
- **Python** (implementation environment for prerank scoring, GSEA, and aggregation of the 26-cell-type count tables)

## Examples

```
import gseapy; gseapy.prerank(rnk=gene_scores, gene_sets='panglaodb', outdir=None)
```

## Evaluation signals

- Enrichment count table is produced for all 26 cell types without missing entries
- A network is judged cell-type-enriched when its cell type name is manually identified in the top 10 cell types of the enrichment results
- Per-gene scores are computed over the full prerank background (all genes in the network set), not a subset
- Consistent score derivation: row-wise (Target-Hub) or column-wise (Regulator-Hub) edge-score sums or NetworkX eigenvector centrality, applied uniformly across methods
- Method-level comparison reproduces the paper's qualitative ordering: scPRINT and scGPT show strong enrichment while GENIE3, DeepSEM, and Geneformer v2 networks are not enriched

## Limitations

- The cell-type-enrichment judgment relies on manually identifying the correct cell type name in the top 10, which is subjective rather than a strict statistical threshold
- No perfect ground truths exist for gene network inference benchmarking, so enrichment-based validation is an indirect biological-meaningfulness check
- Results depend on the choice of node-scoring scheme (Target-Hub vs. Regulator-Hub vs. eigenvector centrality)

## Evidence

- [discussion] BenGRN and GRnnData as benchmarking suite: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
- [intro] benchmark scope on cell type-specific networks: "We extensively benchmark scPRINT on challenging GN inference tasks, from literature-based networks to cell type-specific ones generated via orthogonal sequencing methods."
- [discussion] no perfect ground truths: "A challenging aspect of GN inference is that no perfect ground truths exist"
