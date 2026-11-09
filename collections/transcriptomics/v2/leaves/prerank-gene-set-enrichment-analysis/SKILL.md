---
name: prerank-gene-set-enrichment-analysis
description: Use when you have a weighted gene network or community of genes (e.g.,
  a GRnnData network across the 26 benchmarked cell types or a Louvain community)
  and need to test whether its genes are enriched for TFs, cell-type markers, ENCODE
  targets, or GO molecular-function terms.
when_to_use_negative:
- When the input is raw expression data rather than an already-inferred weighted gene
  network — build the network first (e.g., with scPRINT's attention-matrix extraction)
  before ranking genes
- When the gene set of interest is smaller than the retained-community window or lacks
  a ranked score (prerank requires a full ordered gene list as background)
- When seeking causal ground-truth validation — the article notes no perfect ground
  truths exist for gene network inference benchmarking, so enrichment is a consistency
  check, not causal proof
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0605
tools:
- name: scPRINT
  role: Source of the inferred cell-type-specific gene networks (e.g., BPH vs normal
    fibroblast networks over the 4000 most variable genes) whose node scores are ranked
    for enrichment
  repo: https://github.com/cantinilab/scPRINT
- name: GRnnData
  role: Data structure for loading gene networks and deriving per-gene ordered scores
    via row-wise (Target-Hub) and column-wise (Regulator-Hub) edge-score sums
  repo: https://github.com/cantinilab/GRnnData
- name: BenGRN
  role: Benchmarking suite providing the prerank setup for converting network edge
    scores into ranked gene lists for enrichment testing
  repo: https://github.com/jkobject/benGRN
- name: Python
  role: 'Implementation environment: NetworkX eigenvector centrality, Louvain community
    detection, and prerank Enrichr/GSEA calls'
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:0b10a38bd8de35c4539d3361016b7d35fe9f7b503e7361777df2a347579fb8fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - BenGRN
  - GRnnData
  - Python
  - scPRINT
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# prerank-gene-set-enrichment-analysis

## Summary

Runs GSEA-style prerank enrichment (Enrichr/GSEA) on per-gene ranked scores derived from single-cell gene networks — e.g., eigenvector centrality or Target-Hub/Regulator-Hub edge-score sums — against TF, pangaloDB cell-type marker, ENCODE TF-target, and GO_MF_2023 gene sets to validate biological meaning of scPRINT, scGPT, GENIE3, DeepSEM, and Geneformer networks.

## When to use

Use when you have a weighted gene network or community of genes (e.g., a GRnnData network across the 26 benchmarked cell types or a Louvain community) and need to test whether its genes are enriched for TFs, cell-type markers, ENCODE targets, or GO molecular-function terms.

## When NOT to use

- When the input is raw expression data rather than an already-inferred weighted gene network — build the network first (e.g., with scPRINT's attention-matrix extraction) before ranking genes
- When the gene set of interest is smaller than the retained-community window or lacks a ranked score (prerank requires a full ordered gene list as background)
- When seeking causal ground-truth validation — the article notes no perfect ground truths exist for gene network inference benchmarking, so enrichment is a consistency check, not causal proof

## Inputs

- Inferred cell-type-specific gene networks for each benchmarked method (scPRINT, scGPT, Geneformer, GENIE3, DeepSEM) across 26 cell types, loaded as GRnnData objects
- Per-gene score vectors: row-wise (Target-Hub) or column-wise (Regulator-Hub) edge-score sums, or NetworkX eigenvector centrality
- Gene-set collections: transcription factor sets, pangaloDB cell-type marker sets, ENCODE TF-target sets, GO_MF_2023
- Louvain community gene memberships (resolution 1.5, retained communities of 20–200 genes)

## Outputs

- Prerank GSEA/Enrichr enrichment results per method, cell type, and gene-set collection
- Count tables aggregating enrichment outcomes (e.g., whether the correct cell type appears in the top 10 enriched cell types)
- Community enrichment term lists (e.g., ion-exchange/ion-channel and senescence/inflammation GO molecular-function terms for community 4)
- Collated results file, e.g., bph_network_analysis_results.csv with centrality tables, community memberships, and Enrichr terms

## How to apply

First convert the inferred network into per-gene ordered scores using the BenGRN/GRnnData prerank setup: sum edge scores row-wise (Target-Hub) or column-wise (Regulator-Hub), or compute eigenvector centrality with NetworkX, using all genes in the set as the prerank background. Then run GSEA prerank on these node scores — against TF gene sets to test TF enrichment per cell type, against pangaloDB cell-type marker gene sets to test whether each network matches its cell type, and against each TF's ENCODE-validated target set after ranking the TF's connection scores. For community-level interpretation (e.g., BPH fibroblast community 4 with 92 genes, found via Louvain detection at resolution 1.5 keeping communities of 20–200 genes), rank community genes by eigenvector centrality and run prerank Enrichr enrichment against GO_MF_2023 gene sets. Aggregate results per method and cell type into count tables; per the paper's criterion, a network is judged cell-type-enriched when its cell type name appears manually in the top 10 enriched cell types. Rationale: prerank enrichment uses only the ordering of gene scores, so it directly interrogates whether network hub structure recapitulates known biology (TFs as hubs, correct cell-type markers, ENCODE-validated regulation).

## Related tools

- **scPRINT** (Source of the inferred cell-type-specific gene networks (e.g., BPH vs normal fibroblast networks over the 4000 most variable genes) whose node scores are ranked for enrichment) — https://github.com/cantinilab/scPRINT
- **GRnnData** (Data structure for loading gene networks and deriving per-gene ordered scores via row-wise (Target-Hub) and column-wise (Regulator-Hub) edge-score sums) — https://github.com/cantinilab/GRnnData
- **BenGRN** (Benchmarking suite providing the prerank setup for converting network edge scores into ranked gene lists for enrichment testing) — https://github.com/jkobject/benGRN
- **Python** (Implementation environment: NetworkX eigenvector centrality, Louvain community detection, and prerank Enrichr/GSEA calls)

## Examples

```
import gseapy as gp; res = gp.prerank(rnk=gene_scores.sort_values(ascending=False).to_frame(), gene_sets='GO_MF_2023', outdir=None)
```

## Evaluation signals

- Enrichment count table is produced for all 26 cell types; each network is judged cell-type-enriched only when its cell type name is manually identified in the top 10 enriched cell types
- Retained Louvain communities must fall in the 20–200 gene size window at resolution 1.5 (communities 4 and 5 in BPH-associated fibroblasts)
- Community 4 enrichment (92 genes) should recover calcium/sodium/iron/metal-transport GO_MF_2023 terms, consistent with SPOCK3 and HERC3 hub roles
- Method-level expectation: ~20% of TFs with ENCODE data have connections significantly enriched for their ENCODE-validated targets in scPRINT networks, outperforming all other benchmarked methods
- Sanity check on rankings: scGPT and scPRINT show strong TF node enrichment while GENIE3, DeepSEM, and Geneformer v2 networks are not enriched for TFs

## Limitations

- No perfect ground truths exist for gene network inference benchmarking, and ChIP-seq, perturb-seq, and literature-based ground truths remain scarce and ambiguous
- Cell-type enrichment is judged by manual identification of the cell type name in the top 10 enriched results, a partially subjective criterion
- Community retention (20–200 genes) and Louvain resolution 1.5 are analysis-specific choices that may exclude biologically relevant subnetworks

## Evidence

- [other] BenGRN prerank setup: [evidence span withheld — anchor status: not_found]
- [other] Enrichr GO_MF_2023 community enrichment: [evidence span withheld — anchor status: not_found]
- [other] top-10 cell-type criterion: [evidence span withheld — anchor status: not_found]
- [discussion] BenGRN/GRnnData suite: "With BenGRN and GRnnData, our suite of tools for benchmarking Gene Networks inferred from single-cell RNA sequencing"
- [abstract] BPH ion-exchange finding motivating enrichment: "scPRINT highlights the profound connections between ion exchange, senescence, and chronic inflammation"
