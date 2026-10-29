---
name: go-semantic-similarity-network-construction
description: Use when you have a list of predicted modification-site genes (e.g. 2441
  genes with Ψ sites at >0.99 precision) and need over-representation enrichment plus
  a similarity-scored GO network to expose functional modules such as nucleotide/protein
  binding or neurodegeneration pathways.
when_to_use_negative:
- The prediction task itself (identifying Ψ sites in sequence/structure data) — use
  MU-PseUDeep instead; this skill only analyzes the downstream gene list
- The input is a raw sequence or transcriptome FASTA rather than a curated gene list
  — first run the transcriptome scan and threshold at precision >0.99
- Comparing predictor performance (accuracy, F1, MCC, sensitivity) against XG-PseU,
  PseUI, or iRNA-PseU — this skill does not perform model benchmarking
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: clusterProfiler
  role: Performs GO, pathway, and disease over-representation enrichment on the predicted
    Ψ-site gene list with the human protein-coding transcriptome as background
- name: GOSemSim
  role: Computes pairwise GO semantic similarity scores between enriched genes to
    weight network edges
- name: Cytoscape
  role: Constructs and visualizes the gene network from the semantic similarity scores
- name: ClueGO
  role: Cytoscape plugin used to build the GO semantic-similarity network of enriched
    genes
- name: MU-PseUDeep
  role: Upstream generator of the predicted Ψ-site gene list (human transcriptome
    scan at >0.99 precision) that serves as input to this skill
  repo: https://github.com/smk5g5/MU-PseUDeep
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:297afe6c39e0d5c2ad0b130156cd1bf65da54c6545e7213796e46d4277629610
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smk5g5/MU-PseUDeep
  tools:
  - ClueGO
  - Cytoscape
  - GOSemSim
  - MU-PseUDeep
  - clusterProfiler
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# go-semantic-similarity-network-construction

## Summary

Builds a gene functional network where edges are GO semantic similarity scores computed between genes carrying predicted pseudouridine (Ψ) sites, after clusterProfiler over-representation enrichment against the human protein-coding transcriptome background. Used in MU-PseUDeep to visualize functional relatedness of the 2441 predicted Ψ-site genes.

## When to use

Use when you have a list of predicted modification-site genes (e.g. 2441 genes with Ψ sites at >0.99 precision) and need over-representation enrichment plus a similarity-scored GO network to expose functional modules such as nucleotide/protein binding or neurodegeneration pathways.

## When NOT to use

- The prediction task itself (identifying Ψ sites in sequence/structure data) — use MU-PseUDeep instead; this skill only analyzes the downstream gene list
- The input is a raw sequence or transcriptome FASTA rather than a curated gene list — first run the transcriptome scan and threshold at precision >0.99
- Comparing predictor performance (accuracy, F1, MCC, sensitivity) against XG-PseU, PseUI, or iRNA-PseU — this skill does not perform model benchmarking

## Inputs

- List of genes containing predicted Ψ sites (from Supplemental_table2.csv, human transcriptome scan output)
- Human protein-coding transcriptome gene list (enrichment background)
- GO ontology annotations (Gene Ontology, for clusterProfiler and GOSemSim)

## Outputs

- GO / pathway / disease over-representation results (e.g. 'guanyl-nucleotide exchange factor activity', 'DNA-binding', 'Protein-binding')
- Pairwise GO semantic similarity score matrix for enriched genes
- Cytoscape/ClueGO gene network whose edges are GO semantic similarity scores

## How to apply

Load the gene list (genes containing predicted Ψ sites from the human transcriptome scan, e.g. Supplemental_table2.csv) and run GO, pathway, and disease over-representation enrichment with clusterProfiler, using the human protein-coding transcriptome as the background universe. Then compute pairwise GO semantic similarity between the enriched genes using GOSemSim, so each gene pair receives a semantic similarity score. Import the resulting score matrix into Cytoscape with the ClueGO plugin, constructing a network in which each edge encodes the GO semantic similarity score between two genes. Validate the result by confirming that enriched terms include nucleotide and protein-binding GO categories and neurodegeneration-related pathways, as reported in the paper.

## Related tools

- **clusterProfiler** (Performs GO, pathway, and disease over-representation enrichment on the predicted Ψ-site gene list with the human protein-coding transcriptome as background)
- **GOSemSim** (Computes pairwise GO semantic similarity scores between enriched genes to weight network edges)
- **Cytoscape** (Constructs and visualizes the gene network from the semantic similarity scores)
- **ClueGO** (Cytoscape plugin used to build the GO semantic-similarity network of enriched genes)
- **MU-PseUDeep** (Upstream generator of the predicted Ψ-site gene list (human transcriptome scan at >0.99 precision) that serves as input to this skill) — https://github.com/smk5g5/MU-PseUDeep

## Examples

```
Rscript -e 'ego <- enrichGO(gene = psi_genes, OrgDb = org.Hs.eg.db, universe = pc_genes); sim <- godata("org.Hs.eg.db", computeIC = TRUE)'
```

## Evaluation signals

- Enriched GO categories include nucleotide and protein binding terms (e.g. 'DNA-binding', 'Protein-binding') and neurodegeneration-related pathways, matching the reported finding
- The background universe for enrichment equals the human protein-coding transcriptome, not the whole genome
- Every edge in the Cytoscape/ClueGO network carries a defined GO semantic similarity score between two genes
- Gene list size matches the transcriptome scan output (2441 predicted Ψ-site genes at the >0.99 precision threshold)

## Limitations

- Enrichment is limited to genes meeting the >0.99 precision threshold for Ψ-site prediction, so lower-confidence predictions are excluded
- No experimentally validated overlap for all predictions: only 284 genes had one or more known Ψ site documented in RMBase, so the network reflects predictions rather than validated modifications

## Evidence

- [abstract] Enrichment findings in predicted Ψ-site genes: "the genes with predicted Ψ sites are enriched in nucleotide and protein binding, as well as in neurodegeneration pathways"
- [results] Transcriptome scan that produced the gene list: "We applied MU-PseUDeep to scan the protein-coding genes in the human transcriptome"
- [results] Input file for the gene list: "details about genomic region can be found in the supplemental Excel file, “Supplemental_table2.csv”"
- [results] RMBase-validated overlap caveat: "284 genes already had one or more known Ψ site as documented in the RMBase"
