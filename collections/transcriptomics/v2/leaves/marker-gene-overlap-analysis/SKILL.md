---
name: marker-gene-overlap-analysis
description: Use when a clustering method outputs per-cluster key genes and you must
  verify they are biologically meaningful by testing overlap with published cell-type
  markers and DEGs (|logFC| > 2, adjusted p < 0.01).
when_to_use_negative:
- The clustering outputs are not scQA landmark/key genes (e.g., you only have cluster
  labels without the key genes defining each cluster) — there is nothing to validate
  against the marker/DEG references.
- The dataset's original publication does not provide marker or DEG gene sets to serve
  as reference, leaving no external ground truth for overlap testing.
- You need to identify cell types or cluster cells de novo — use the upstream scQA
  clustering pipeline (pre-processing, LC1/LC2, CC modules) instead of the downstream
  validation step.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: Produces the clustering labels and cell type-specific key genes (landmarks)
    that this validation workflow recovers per cluster and tests against marker/DEG
    reference sets.
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:06de3fb0142ec761339a0292e74e1fb8eb404d798657cdb3d2a85a08256a0ea8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - scQA
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# marker-gene-overlap-analysis

## Summary

Validates scQA key genes (landmark genes) per cluster by computing overlap and enrichment statistics against marker genes and DEGs compiled from the dataset's original publication, confirming biological significance as in the Camp2 endothelial case (ADTRP, CDH5, CLEC14A, ACVRL1).

## When to use

Use when a clustering method outputs per-cluster key genes and you must verify they are biologically meaningful by testing overlap with published cell-type markers and DEGs (|logFC| > 2, adjusted p < 0.01).

## When NOT to use

- The clustering outputs are not scQA landmark/key genes (e.g., you only have cluster labels without the key genes defining each cluster) — there is nothing to validate against the marker/DEG references.
- The dataset's original publication does not provide marker or DEG gene sets to serve as reference, leaving no external ground truth for overlap testing.
- You need to identify cell types or cluster cells de novo — use the upstream scQA clustering pipeline (pre-processing, LC1/LC2, CC modules) instead of the downstream validation step.

## Inputs

- labels.txt clustering output from scQA (chained from the upstream clustering task)
- Preprocessed expression matrix for the dataset (e.g., Camp2 GSE81252) used to recover genes per cluster
- Reference gene sets compiled from the dataset's original publication: cell-type marker genes and DEGs (|logFC| > 2, adjusted p-value < 0.01)

## Outputs

- key_genes.tsv: per-cluster key-gene table with overlap counts and hypergeometric/Fisher-style enrichment statistics against marker and DEG reference sets

## How to apply

Load the scQA clustering output labels.txt (chained from a prior clustering run) together with the preprocessed cells-by-genes expression matrix to recover the genes defining each cluster as key genes (consensus landmarks). Compile reference gene sets from the dataset's original publication: cell-type marker genes and DEGs selected with absolute logFC above 2 and adjusted p-value less than 0.01. For each cluster, compute overlap counts and hypergeometric/Fisher-style enrichment between its key genes and the marker/DEG reference sets, then check coherence against paper-reported examples (e.g., endothelial key genes such as ADTRP, CDH5, CLEC14A, ACVRL1 associated with blood vessel development; note ADTRP instead regulates endothelial proliferation and migration). Write the per-cluster table to key_genes.tsv, accepting it only when each cluster's key genes show documented overlap with known markers/DEGs consistent with external and internal key-gene validation.

## Related tools

- **scQA** (Produces the clustering labels and cell type-specific key genes (landmarks) that this validation workflow recovers per cluster and tests against marker/DEG reference sets.) — https://github.com/LD-Lyndee/scQA

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git && ./main xin_matrix.txt o  # then recover per-cluster key genes from labels.txt and test overlap vs. marker/DEG sets into key_genes.tsv
```

## Evaluation signals

- key_genes.tsv exists with one row-set per cluster and columns for overlap counts and enrichment statistics against both marker and DEG reference sets.
- Each cluster's key genes show documented overlap with known markers/DEGs, matching the paper's external/internal key-gene validation.
- Endothelial key genes (e.g., in Camp2, LM2) are enriched in endothelial-related GO terms such as blood vessel development, with CDH5, CLEC14A, ACVRL1 as endothelial markers and ADTRP interpretable as a regulator of endothelial proliferation and migration.
- Reference DEG lists were built with the paper's exact thresholds: |logFC| > 2 and adjusted p-value < 0.01.
- Enrichment p-values are computed with a defensible hypergeometric/Fisher-style test on the overlap of each cluster's key genes with the reference sets.

## Limitations

- Key genes depend on upstream scQA limits: retaining only 2000 HVGs excluded some informative genes, and genes may be misclassified in LC1 and LC2 due to high dimensionality.
- Some clusters may lack marker support entirely (e.g., Cluster8 in the Xenopus lung dataset contained only three cells, likely outliers), yielding no interpretable marker overlap.
- Mixed clusters can blur validation: endothelial and mesenchymal cells were incorrectly grouped into a single cluster (Cluster3), complicating per-cluster marker interpretation.

## Evidence

- [other] the landmark genes scQA identified for the endothelial cell type (e.g., LM2 in the Camp2 dataset) are enriched in endothelial-related GO terms: [evidence span withheld — anchor status: not_found]
- [results] DEG selection criteria quote: "genes with the absolute value of logFC above 2 and adjusted p-value less than 0.01 were selected as DEGs"
- [intro] key genes share attributes with DEGs, hub and marker genes: "scQA captures key genes sharing attributes with differentially expressed genes, hub genes and marker genes, serving as reliable landmarks for clustering cells"
- [abstract] external and internal validation of key genes: "we confirm the effectiveness and potential biological significance of the identified key genes through both external and internal validation"
- [readme] README output labels.txt: "- **labels.txt** Clustering result."
