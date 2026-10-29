---
name: cell-type-marker-enrichment-annotation
description: Use when scQA clustering (labels.txt) has assigned cells to clusters
  and you must confirm each cluster's key genes overlap known markers, hub genes,
  and DEGs (e.g., endothelial ADTRP, CDH5, CLEC14A, ACVRL1).
when_to_use_negative:
- Clustering labels have not yet been generated (labels.txt from scQA is a prerequisite;
  chain from the clustering task first)
- The dataset's original publication does not provide marker genes or DEG lists, so
  no reference sets exist to compute overlap against
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: Produces the clustering labels and cell type-specific key genes (landmarks)
    whose biological significance is validated against markers and DEGs
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:58fbc438f0133ed75d1da498622f8dfc717a6f600fab66f6dd803a799502f6fe
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
yes_rate: 0.6666666666666666
n_claims: 12
---

# cell-type-marker-enrichment-annotation

## Summary

Validates the biological significance of key genes (landmarks) identified by scQA for each cell-type cluster by computing their overlap and enrichment against reference marker genes and DEGs (|logFC| > 2, adjusted p < 0.01) from the dataset's original publication.

## When to use

Use when scQA clustering (labels.txt) has assigned cells to clusters and you must confirm each cluster's key genes overlap known markers, hub genes, and DEGs (e.g., endothelial ADTRP, CDH5, CLEC14A, ACVRL1).

## When NOT to use

- Clustering labels have not yet been generated (labels.txt from scQA is a prerequisite; chain from the clustering task first)
- The dataset's original publication does not provide marker genes or DEG lists, so no reference sets exist to compute overlap against

## Inputs

- labels.txt clustering output from scQA run (chained from task_004)
- preprocessed expression matrix (cells * genes, numeric, no row/column names) for the dataset
- reference gene sets from the dataset's original publication: cell-type marker genes and DEGs (|logFC| > 2, adjusted p < 0.01)

## Outputs

- key_genes.tsv: per-cluster key-gene table with marker/DEG overlap counts and enrichment statistics

## How to apply

Load the scQA clustering output labels.txt (chained from the clustering run on the preprocessed cells-by-genes matrix) and recover the genes defining each cluster as key genes (consensus landmarks). Compile reference gene sets from the dataset's original publication: cell-type marker genes and DEGs selected with |logFC| > 2 and adjusted p-value < 0.01. For each cluster, compute overlap statistics (overlap counts plus hypergeometric/Fisher-style enrichment) between its key genes and the marker/DEG reference sets. Check key-gene coherence against paper-reported examples, e.g., endothelial key genes (ADTRP, CDH5, CLEC14A, ACVRL1 in Camp2 GSE81252) enriched in blood vessel development GO terms. Write the per-cluster key-gene table with overlap and enrichment statistics to key_genes.tsv.

## Related tools

- **scQA** (Produces the clustering labels and cell type-specific key genes (landmarks) whose biological significance is validated against markers and DEGs) — https://github.com/LD-Lyndee/scQA

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git && cd scQA && ./make && ./main camp2_matrix.txt o  # then overlap cluster key genes from labels.txt vs paper DEGs (|logFC|>2, padj<0.01) -> key_genes.tsv
```

## Evaluation signals

- Each cluster's key genes show documented overlap with known markers/DEGs (hypergeometric/Fisher-style enrichment significant)
- Endothelial key genes (e.g., LM2 in Camp2) are enriched in endothelial-related GO terms such as blood vessel development
- Identified key genes include paper-reported markers: ADTRP (regulates endothelial proliferation and migration), CDH5, CLEC14A, ACVRL1
- key_genes.tsv contains one row-set per cluster with overlap counts and enrichment statistics populated
- Landmark count per cluster remains small (approximately a dozen), consistent with scQA's iterative minimization of landmarks

## Limitations

- Retention of only 2000 highly variable genes during pre-processing excluded some informative genes, so key-gene recall is incomplete
- Genes may be misclassified in LC1 and LC2 due to high dimensionality, potentially yielding spurious landmarks
- Some clusters (e.g., Cluster8 in the Xenopus lung dataset with only three cells) lack marker genes and likely represent outliers; also, endothelial and mesenchymal cells were incorrectly grouped into a single Cluster3 in one dataset

## Evidence

- [abstract] Effectiveness and potential biological significance of identified key genes were confirmed through external and internal validation: "we confirm the effectiveness and potential biological significance of the identified key genes through both external and internal validation"
- [results] DEG selection thresholds: "genes with the absolute value of logFC above 2 and adjusted p-value less than 0.01 were selected as DEGs"
- [intro] Key genes as landmarks shared with markers/hubs/DEGs: "scQA captures key genes sharing attributes with differentially expressed genes, hub genes and marker genes, serving as reliable landmarks for clustering cells"
- [readme] Output format of scQA run: "- **labels.txt** Clustering result."
- [results] Endothelial key genes validated against blood vessel development GO terms: [evidence span withheld — anchor status: not_found]
