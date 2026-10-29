---
name: cell-communication-gene-labeling
description: Use when an HVG list (e.g. from FindVariableFeatures(), variance, scannpy,
  or M3drop on seqFISH+/Seq-Scope data) must be split into communication-related (Y)
  vs non-related (N) genes via GeneCards/GO labels.
when_to_use_negative:
- When the goal is de novo discovery of a communication-gene classification criterion
  — the article notes existing methods 'lack a standardized classification criterion',
  and this skill only applies predefined GeneCards/GO labels
- When gene lists already carry validated communication annotations and only need
  downstream MIMO network construction (start instead from the directed-graph/MIMO
  step)
- When the organism's genes have no GeneCards/GO coverage, since label extraction
  depends entirely on those databases
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GeneCards
  role: Database queried for gene labels ('receptor', 'ligand', 'cell–cell adhesion',
    'intercellular interaction') used as the strong-correlation classification source
- name: Gene Ontology (GO)
  role: Second predefined-label database used alongside GeneCards to classify genes
    into the two communication classes
- name: CPPLS-MLP
  role: Provides the search_gene code that applies the communication criteria to an
    input gene list, and the compare_hvg.zip HVG lists being labeled
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Python
  role: Runtime for the search_gene/divide classification scripts (Python >= 3.7,
    networkx 3.2.1)
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:337f2f36fd044a490b7fe822fbcfcac9df24789e600d9605a25ecf70fd3b3a9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CCPLS
  - CPPLS-MLP
  - Gene Ontology (GO)
  - GeneCards
  - Python
  - R
  - Seurat
  - pls
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cell-communication-gene-labeling

## Summary

Labels highly variable genes (HVGs) as cell-communication-related or not, by querying GeneCards/Gene Ontology for predefined labels (receptor, ligand, cell–cell adhesion, intercellular interaction) and assigning binary strong_cor values used to filter HVGs in CPPLS-MLP analyses.

## When to use

Use when an HVG list (e.g. from FindVariableFeatures(), variance, scannpy, or M3drop on seqFISH+/Seq-Scope data) must be split into communication-related (Y) vs non-related (N) genes via GeneCards/GO labels.

## When NOT to use

- When the goal is de novo discovery of a communication-gene classification criterion — the article notes existing methods 'lack a standardized classification criterion', and this skill only applies predefined GeneCards/GO labels
- When gene lists already carry validated communication annotations and only need downstream MIMO network construction (start instead from the directed-graph/MIMO step)
- When the organism's genes have no GeneCards/GO coverage, since label extraction depends entirely on those databases

## Inputs

- HVG lists extracted with FindVariableFeatures(), variance, scannpy, and M3drop (precomputed in compare_hvg.zip; 16,000 HVGs total)
- Gene identifier lists (e.g. Y HVGs and N HVGs per cell type cluster)
- Access to GeneCards and Gene Ontology (GO) databases for label querying

## Outputs

- Binary strong_com label vector (1 for communication-labeled genes, 0 for unlabeled) with weak-correlation labels marked separately
- Per-cell-type sets of Y (communication-related) and N (non-related) HVGs
- Aggregated stability/overlap metrics (e.g. ccpls_mlp_hvg_stability_comparison.csv with method and stability score columns)

## How to apply

Query GeneCards and Gene Ontology for each gene in the HVG list, extracting labels such as 'receptor', 'ligand', 'receptors', 'ligands', 'cell–cell adhesion', and 'intercellular interaction'. Assign value 1 (strong_com) to genes carrying a strong-correlation label and 0 to unlabeled genes, marking weak-correlation labels separately. In the CPPLS-MLP repository, use the provided search_gene code: enter the gene list and run it to apply the authors' communication criteria across all genes. The resulting labels filter out HVGs not involved in cell communication before constructing MIMO networks, and are used to compute per-cell-type Y-HVG percentages and Y-HVG overlap rates between cell types (strong vs weak labels) as stability scores.

## Related tools

- **GeneCards** (Database queried for gene labels ('receptor', 'ligand', 'cell–cell adhesion', 'intercellular interaction') used as the strong-correlation classification source)
- **Gene Ontology (GO)** (Second predefined-label database used alongside GeneCards to classify genes into the two communication classes)
- **CPPLS-MLP** (Provides the search_gene code that applies the communication criteria to an input gene list, and the compare_hvg.zip HVG lists being labeled) — https://github.com/wuzhenao/CPPLS-MLP
- **Python** (Runtime for the search_gene/divide classification scripts (Python >= 3.7, networkx 3.2.1)) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
R> search_gene(gene_list)  # enter the gene list and run search_gene from the CPPLS-MLP repo (compare_hvg.zip code) to apply the communication criteria
```

## Evaluation signals

- Every gene carries a binary strong_com value in {0,1}, with weak-correlation labels recorded separately from strong ones
- After labeling and filtering, CPPLS-MLP identified 354 (seqFISH+) and 345 (Seq-Scope) communication-related genes — counts should be in this range on these datasets
- Aggregated Y-HVG overlap rates must lie in [0,1] and match reported values: 0.31 (Seq-Scope) and 0.25 (seqFISH+)
- The output stability table (ccpls_mlp_hvg_stability_comparison.csv) contains method and stability-score fields with exactly 2 method rows per dataset
- search_gene returns a communication classification for every gene in the supplied list, not only HVGs

## Limitations

- Label extraction is a manual database-querying process with no scripted or automated procedure described in the article
- There is no standardized classification criterion for whether a gene is associated with cell communication; these labels are one heuristic set from GeneCards/GO
- Only two benchmark datasets (seqFISH+ mouse cortex, Seq-Scope mouse colon) were used to assess the labeling downstream, limiting generality

## Evidence

- [methods] categorized into two classes based on predefined labels from databases such as GeneCards and Gene Ontology (GO): "categorized into two classes based on predefined labels from databases such as GeneCards and Gene Ontology (GO)"
- [discussion] labels genes in a known database to filter out HVGs that are not involved in cell communication: "labels genes in a known database to filter out HVGs that are not involved in cell communication"
- [discussion] By manually querying genes in databases like Genecard and GO and extracting labels: "By manually querying genes in databases like Genecard and GO and extracting labels"
- [readme] Enter the list of genes and then run the search_gene to get the results under this criterion.: "Enter the list of genes and then run the search_gene to get the results under this criterion."
- [methods] our CPPLS-MLP method identified 354 and 345 genes related to cell communication, respectively: "our CPPLS-MLP method identified 354 and 345 genes related to cell communication, respectively"
- [abstract] which can identify HVGs closely related to intercellular communication: "which can identify HVGs closely related to intercellular communication"
