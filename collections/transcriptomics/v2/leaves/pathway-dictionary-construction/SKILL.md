---
name: pathway-dictionary-construction
description: Use when preparing pathway prior knowledge as input to scPAFA's fast_Ucell
  or fast_score_genes PAS computation, i.e., before Step1 'Calculate Pathway Activity
  Score'.
when_to_use_negative:
- 'The analysis goal is de novo gene module discovery without prior knowledge — scPAFA''s
  pathway input is supervised: ''the selection of pathways is a supervised process,
  where using a redundant pathway set may increase the complexity of the results'''
- Input is already an scPAFA-ready pathway dictionary of pathway name → gene list
  mappings
- The downstream tool does not accept gene-set dictionaries (e.g., methods requiring
  GMT pre-processing internally or sample-level expression matrices only)
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: scPAFA
  role: Consumes the pathway dictionary as input to fast_Ucell / fast_score_genes
    PAS computation; provides the generate_pathway_input.ipynb tutorial
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: MsigDB
  role: Source of downloadable pathway collections (JSON bundle recommended) used
    to build the dictionary
- name: NCATS BioPlanet
  role: Source of the manually curated human pathway set (1658 pathways) primarily
    used with scPAFA in the paper
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:a7d932c8227d42c3b3e14919ef17706d896add223dee3f434e094e16898f5ce6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - AUCell
  - MsigDB
  - NCATS BioPlanet
  - Python
  - UCell
  - scPAFA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# pathway-dictionary-construction

## Summary

Construct the Python pathway dictionary required by scPAFA: each item maps a pathway name (key) to a list of genes (values), built from public collections such as MsigDB or NCATS BioPlanet, or from a custom pathway set.

## When to use

Use when preparing pathway prior knowledge as input to scPAFA's fast_Ucell or fast_score_genes PAS computation, i.e., before Step1 'Calculate Pathway Activity Score'.

## When NOT to use

- The analysis goal is de novo gene module discovery without prior knowledge — scPAFA's pathway input is supervised: 'the selection of pathways is a supervised process, where using a redundant pathway set may increase the complexity of the results'
- Input is already an scPAFA-ready pathway dictionary of pathway name → gene list mappings
- The downstream tool does not accept gene-set dictionaries (e.g., methods requiring GMT pre-processing internally or sample-level expression matrices only)

## Inputs

- MsigDB pathway collection (JSON bundle recommended, e.g. H collection from gsea-msigdb.org)
- NCATS BioPlanet pathway table (pathway.csv from tripod.nih.gov/bioplanet)
- Optional: custom pathway collection supplied by the user

## Outputs

- Python dictionary with pathway names as keys and lists of genes as values, suitable for scPAFA fast_Ucell / fast_score_genes input

## How to apply

Download a pathway collection — the recommended option is the MsigDB 'JSON bundle' from gsea-msigdb.org, or the NCATS BioPlanet pathway.csv from tripod.nih.gov/bioplanet — or supply a custom pathway collection. Parse the downloaded file and reshape it into a Python dictionary where each key is a pathway name and each value is the list of genes in that pathway. scPAFA provides a tutorial notebook (tutorial/generate_pathway_input.ipynb) demonstrating construction from both MsigDB and NCATS BioPlanet; the paper primarily used NCATS BioPlanet, an integrated resource of 1658 manually curated human pathways, and benchmarks also used 1629 pathways (CRC) and 1383 pathways (lupus). Validate the dictionary by confirming each pathway maps to a gene list that can be intersected with the gene names of the scRNA-seq expression matrix, since these dictionaries feed the ranking-based PAS implementations.

## Related tools

- **scPAFA** (Consumes the pathway dictionary as input to fast_Ucell / fast_score_genes PAS computation; provides the generate_pathway_input.ipynb tutorial) — https://github.com/ZhuoliHuang/scPAFA
- **MsigDB** (Source of downloadable pathway collections (JSON bundle recommended) used to build the dictionary)
- **NCATS BioPlanet** (Source of the manually curated human pathway set (1658 pathways) primarily used with scPAFA in the paper)

## Examples

```
from scPAFA.tutorial.generate_pathway_input import build_pathway_dict; pathway_dict = build_pathway_dict('pathway.csv')  # per tutorial/generate_pathway_input.ipynb
```

## Evaluation signals

- The resulting object is a Python dict whose keys are pathway names and whose values are lists of genes (per README: 'each item with a pathway name as a key and a list of genes as values')
- Pathway counts match the source collection scale — e.g., 1658 NCATS BioPlanet pathways, or the paper's benchmark sets of 1629 (CRC) and 1383 (lupus) pathways
- Pathway gene lists are intersectable with the gene names in the single-cell expression matrix (no entirely unmatched pathways feeding Step1)
- Output matches the format produced by the reference tutorial notebook tutorial/generate_pathway_input.ipynb in the scPAFA repository

## Limitations

- Pathway selection is a supervised process: a redundant pathway set may increase complexity of downstream MOFA results, while too few or biologically misaligned pathways may cause information loss
- The dictionary should be tailored to the biological context of the study; the article cautions that 'choosing too few or pathways that do not align with the biological context may cause information loss'

## Evidence

- [readme] Pathway input: Download pathway information and generate a pathway dictionary — The pathway input of scPAFA is a Python dictionary, each item with a pathway name as a key and a list of genes as values.: "The pathway input of scPAFA is a Python dictionary, each item with a pathway name as a key and a list of genes as values."
- [readme] README sources for pathway collections: "Pathway collection can be downloaded from [MsigDB](https://www.gsea-msigdb.org/gsea/msigdb/human/collections.jsp#H) ('JSON bundle' is recommended), or [NCATS"
- [discussion] Paper on BioPlanet as primary collection: "we primarily used NCATS BioPlanet 27 with scPAFA, an integrated resource of 1658 manually curated human pathways."
- [discussion] Paper on supervised pathway selection caveat: "the selection of pathways is a supervised process, where using a redundant pathway set may increase the complexity of the results"
