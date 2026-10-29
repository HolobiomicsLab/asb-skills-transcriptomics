---
name: pathway-and-disease-enrichment-analysis
description: Use when you have a list of genes carrying predicted Ψ sites (e.g. from
  a >0.99-precision transcriptome scan in Supplemental_table2.csv) and need to identify
  which molecular functions, pathways, or diseases are functionally over-represented.
when_to_use_negative:
- The input is raw uridine-centered RNA fragments or per-site predictions rather than
  a gene-level list suitable for over-representation analysis
- The analysis target is a non-human organism or non-protein-coding transcriptome
  — the paper's enrichment was run against the human protein-coding transcriptome
  background only
- Predicted Ψ sites below the >0.99 precision threshold are included, since the enrichment
  analysis was defined on the high-precision gene set
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0091
tools:
- name: MU-PseUDeep
  role: Upstream predictor that generated the Ψ-site gene list (>0.99 precision) scanned
    over the human protein-coding transcriptome
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: clusterProfiler
  role: Performs GO, pathway, and disease over-representation enrichment on the predicted
    Ψ-site gene list
- name: GOSemSim
  role: Computes pairwise GO semantic similarity scores between the enriched genes
    for network weighting
- name: Cytoscape
  role: Platform in which the gene network is constructed and visualized
- name: ClueGO
  role: Cytoscape plugin used to build the gene network where each edge represents
    the GO semantic similarity score between two genes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:640855d21218a569b1382d99e10c8930d8c2d096ec7f5adacfbd4c2dd7b92930
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pathway-and-disease-enrichment-analysis

## Summary

Over-representation analysis of GO terms, pathways, and diseases for the 2441 human genes containing MU-PseUDeep-predicted pseudouridine (Ψ) sites, followed by GO semantic similarity network construction, to reveal that predicted Ψ-site genes are enriched in nucleotide/protein binding and neurodegeneration pathways.

## When to use

Use when you have a list of genes carrying predicted Ψ sites (e.g. from a >0.99-precision transcriptome scan in Supplemental_table2.csv) and need to identify which molecular functions, pathways, or diseases are functionally over-represented.

## When NOT to use

- The input is raw uridine-centered RNA fragments or per-site predictions rather than a gene-level list suitable for over-representation analysis
- The analysis target is a non-human organism or non-protein-coding transcriptome — the paper's enrichment was run against the human protein-coding transcriptome background only
- Predicted Ψ sites below the >0.99 precision threshold are included, since the enrichment analysis was defined on the high-precision gene set

## Inputs

- List of genes containing predicted Ψ sites from Supplemental_table2.csv (MU-PseUDeep human transcriptome scan, >0.99 precision threshold)
- Human protein-coding transcriptome gene list as background universe
- Gene ontology (GO) annotation database (e.g. org.Hs.eg.db via clusterProfiler)

## Outputs

- Enriched GO molecular function, pathway, and disease over-representation terms (as shown in Fig. 6)
- Pairwise GO semantic similarity score matrix between enriched genes (GOSemSim)
- Cytoscape/ClueGO gene network with edges weighted by GO semantic similarity
- Validation report confirming nucleotide/protein-binding and neurodegeneration pathway enrichments

## How to apply

Load the genes containing predicted Ψ sites that passed the >0.99 precision threshold from the human transcriptome scan output (Supplemental_table2.csv). Run GO, pathway, and disease over-representation enrichment with clusterProfiler, using the human protein-coding transcriptome as the background universe rather than the whole genome. Compute pairwise GO semantic similarity scores between the enriched genes using GOSemSim, then build a Cytoscape gene network with the ClueGO plugin in which each edge is weighted by the GO semantic similarity score between two genes. The rationale is that Ψ-site genes are expected to cluster in functional categories, so validating against the paper's reported enrichments (guanyl-nucleotide exchange factor activity, DNA-binding, protein-binding, neurodegeneration pathways) confirms the analysis matches the published finding (Fig. 6).

## Related tools

- **MU-PseUDeep** (Upstream predictor that generated the Ψ-site gene list (>0.99 precision) scanned over the human protein-coding transcriptome) — https://github.com/smk5g5/MU-PseUDeep
- **clusterProfiler** (Performs GO, pathway, and disease over-representation enrichment on the predicted Ψ-site gene list)
- **GOSemSim** (Computes pairwise GO semantic similarity scores between the enriched genes for network weighting)
- **Cytoscape** (Platform in which the gene network is constructed and visualized)
- **ClueGO** (Cytoscape plugin used to build the gene network where each edge represents the GO semantic similarity score between two genes)

## Examples

```
R: enrichGO(gene = psi_gene_list, OrgDb = org.Hs.eg.db, universe = protein_coding_transcriptome, ont = "MF", pAdjustMethod = "BH")
```

## Evaluation signals

- Enriched GO terms must include nucleotide and protein binding categories, matching the paper's abstract finding
- Neurodegeneration-related pathways should appear among the enriched pathway results, as reported in the paper
- The specific categories 'guanyl-nucleotide exchange factor activity', 'DNA-binding', and 'Protein-binding' should be recovered (Fig. 6)
- The gene count entering enrichment should be ~2441 predicted Ψ-site genes, filtered at the >0.99 precision threshold
- The Cytoscape network should have edge weights equal to the GOSemSim pairwise GO semantic similarity scores

## Limitations

- Enrichment is limited to predictions above the >0.99 precision threshold, so lower-confidence Ψ sites are excluded from the functional analysis
- Only the human protein-coding transcriptome was used as the background universe; non-coding RNAs are not covered
- Predicted Ψ sites are model-derived (MU-PseUDeep), not experimentally validated, so enrichments reflect prediction bias as well as biology

## Evidence

- [abstract] Enriched GO molecular function terms: "the genes with predicted Ψ sites are enriched in nucleotide and protein binding, as well as in neurodegeneration pathways"
- [methods] Threshold defining the gene set: "A precision threshold of >0.99 was used to predict whether the uridine site is a potential Ψ site"
- [results] Input file for the enrichment gene list: "details about genomic region can be found in the supplemental Excel file, “Supplemental_table2.csv”"
- [results] Transcriptome scan providing enrichment input: "We applied MU-PseUDeep to scan the protein-coding genes in the human transcriptome"
