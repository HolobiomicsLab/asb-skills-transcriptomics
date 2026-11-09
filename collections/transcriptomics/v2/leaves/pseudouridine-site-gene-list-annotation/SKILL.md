---
name: pseudouridine-site-gene-list-annotation
description: Use when you have a list of genes with predicted Ψ sites (e.g., MU-PseUDeep
  human transcriptome scan at >0.99 precision) and need GO/pathway/disease over-representation
  against a protein-coding background.
when_to_use_negative:
- Input is a list of Ψ sites/positions rather than gene identifiers — this skill annotates
  genes, not individual modified nucleotides
- The organism is mouse (mm10) or yeast (sacCer3) rather than human — the validated
  background and validation criteria here are human protein-coding transcriptome specific
- The predicted sites came from a different tool without a defined precision threshold,
  so the gene-list provenance cannot be reproduced
edam_operation: http://edamontology.org/operation_3939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3391
tools:
- name: MU-PseUDeep
  role: Generates the upstream gene list by scanning the human protein-coding transcriptome
    at a >0.99 precision threshold for uridines that are potential Ψ sites
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: clusterProfiler
  role: Performs GO, pathway, and disease over-representation enrichment on the predicted
    Ψ-site gene list against the human protein-coding transcriptome background
- name: GOSemSim
  role: Computes pairwise GO semantic similarity scores between enriched genes
- name: Cytoscape
  role: Hosts the gene network visualization of predicted Ψ-site genes
- name: ClueGO
  role: Cytoscape plugin used to build the network where each edge represents the
    GO semantic similarity score between two genes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:15b9e3d0dc50f2b92f80a9c8e1123ec54f7d26366875f6d5a1af60f15113d286
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
  edam_operation: http://edamontology.org/operation_3939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# pseudouridine-site-gene-list-annotation

## Summary

Annotates the genes carrying predicted pseudouridine (Ψ) sites from a MU-PseUDeep transcriptome scan by running GO, pathway, and disease over-representation enrichment with clusterProfiler, then visualizing GO semantic relationships in Cytoscape/ClueGO. It links deep-learning site predictions to molecular function and disease-level interpretation.

## When to use

Use when you have a list of genes with predicted Ψ sites (e.g., MU-PseUDeep human transcriptome scan at >0.99 precision) and need GO/pathway/disease over-representation against a protein-coding background.

## When NOT to use

- Input is a list of Ψ sites/positions rather than gene identifiers — this skill annotates genes, not individual modified nucleotides
- The organism is mouse (mm10) or yeast (sacCer3) rather than human — the validated background and validation criteria here are human protein-coding transcriptome specific
- The predicted sites came from a different tool without a defined precision threshold, so the gene-list provenance cannot be reproduced

## Inputs

- List of genes containing predicted Ψ sites from MU-PseUDeep human transcriptome scan (Supplemental_table2.csv, >0.99 precision threshold)
- Human protein-coding transcriptome gene set used as enrichment background

## Outputs

- GO, pathway, and disease over-representation results (enriched terms with statistics, e.g., Fig. 6 categories)
- Pairwise GO semantic similarity scores between enriched genes (GOSemSim)
- Gene network in Cytoscape/ClueGO with edges weighted by GO semantic similarity

## How to apply

Load the list of genes containing predicted Ψ sites from the human transcriptome scan output (Supplemental_table2.csv), produced at a precision threshold of >0.99. Run GO, pathway, and disease over-representation enrichment using clusterProfiler, explicitly setting the human protein-coding transcriptome as the background universe so enrichment is not biased by gene length or annotation coverage. Compute pairwise GO semantic similarity scores between the enriched genes with GOSemSim, and build a gene network in Cytoscape with the ClueGO plugin where each edge weight is the GO semantic similarity between two genes. Validate the result by confirming that enriched terms include nucleotide and protein binding GO categories (e.g., 'guanyl-nucleotide exchange factor activity,' 'DNA-binding,' 'Protein-binding') and neurodegeneration-related pathways, matching the reported finding for the 2441 predicted Ψ-site genes.

## Related tools

- **MU-PseUDeep** (Generates the upstream gene list by scanning the human protein-coding transcriptome at a >0.99 precision threshold for uridines that are potential Ψ sites) — https://github.com/smk5g5/MU-PseUDeep
- **clusterProfiler** (Performs GO, pathway, and disease over-representation enrichment on the predicted Ψ-site gene list against the human protein-coding transcriptome background)
- **GOSemSim** (Computes pairwise GO semantic similarity scores between enriched genes)
- **Cytoscape** (Hosts the gene network visualization of predicted Ψ-site genes)
- **ClueGO** (Cytoscape plugin used to build the network where each edge represents the GO semantic similarity score between two genes)

## Examples

```
Rscript -e "library(clusterProfiler); library(org.Hs.eg.db); ego <- enrichGO(gene = psi_genes, universe = pc_genes, OrgDb = org.Hs.eg.db, keyType = 'ENSEMBL', ont = 'BP')"
```

## Evaluation signals

- The number of genes entering the enrichment analysis matches the reported 2441 predicted Ψ-site genes from the human transcriptome scan
- Enriched GO categories include nucleotide and protein binding (e.g., 'guanyl-nucleotide exchange factor activity,' 'DNA-binding,' 'Protein-binding') as reported in Fig. 6
- Neurodegeneration-related pathways appear among enriched terms, consistent with the paper's abstract-level finding
- The enrichment universe is explicitly the human protein-coding transcriptome, not the whole genome or all annotated transcripts
- The ClueGO network edges correspond to GOSemSim semantic similarity scores, and the network is connected/interpretable rather than empty

## Limitations

- Predicted Ψ sites are computational (precision threshold >0.99), not experimentally validated; only 284 genes already had known Ψ sites documented in RMBase
- Enrichment depends on the background choice — using a non-protein-coding or whole-genome background would change the enriched categories
- A direct comparison with iPseU-CNN was impossible because no source code or webserver was available, so the gene list cannot be cross-validated with that tool

## Evidence

- [abstract] enrichment finding: "the genes with predicted Ψ sites are enriched in nucleotide and protein binding, as well as in neurodegeneration pathways"
- [methods] precision threshold: "A precision threshold of >0.99 was used to predict whether the uridine site is a potential Ψ site"
- [results] transcriptome scan input: "We applied MU-PseUDeep to scan the protein-coding genes in the human transcriptome"
- [full_text] workflow details: [evidence span withheld — anchor status: not_found]
