---
name: drug-target-actionability-annotation
description: Use when you have a candidate gene list from a network or biomarker pipeline
  and need to flag which genes are targets of pharmacologically active drugs (DrugBank)
  and which lack prior literature links to a disease.
when_to_use_negative:
- The goal is functional or pathway enrichment of gene communities (use g:Profiler
  GO:BP/KEGG analysis instead), not drug target annotation.
- No DrugBank pharmacologically-active target file is available and downloading/curating
  external knowledge bases is out of scope.
- The input is differential-expression gene counts rather than extracted IC gene symbol
  lists — drug actionability annotation is only meaningful on curated gene symbol
  sets.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0208
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_2640
tools:
- name: Python
  role: Scripted matching of IC gene symbols against the DrugBank pharmacologically
    active target file and compilation of the actionability table
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Hosts 'Knowledge-based evaluation of the results.ipynb', which performs the
    DrugBank evaluation and PubMed literature association
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:8d7c30279e28c7f5cb157d09072be1ffaea5a7b9aff43114afd6cf5fc8b86483
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - Jupyter Notebook
  - Python
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0208
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# drug-target-actionability-annotation

## Summary

Annotates extracted IC gene lists (e.g., from fused KIRC, LIHC, PRAD co-expression networks) as drug actionable by cross-referencing gene symbols against DrugBank pharmacologically-active targets, and combines this with PubMed literature association to flag novel candidate biomarkers.

## When to use

Use when you have a candidate gene list from a network or biomarker pipeline and need to flag which genes are targets of pharmacologically active drugs (DrugBank) and which lack prior literature links to a disease.

## When NOT to use

- The goal is functional or pathway enrichment of gene communities (use g:Profiler GO:BP/KEGG analysis instead), not drug target annotation.
- No DrugBank pharmacologically-active target file is available and downloading/curating external knowledge bases is out of scope.
- The input is differential-expression gene counts rather than extracted IC gene symbol lists — drug actionability annotation is only meaningful on curated gene symbol sets.

## Inputs

- IC gene symbol lists per cancer type from the Extracted folder, files named 'IC_'+str(tumor)+'.csv' (KIRC, LIHC, PRAD)
- DrugBank DB_pharmacologically_active.csv (Supplementary data, pharmacologically active targets)
- PubMed evaluation results from 'Knowledge-based evaluation of the results.ipynb'

## Outputs

- candidate_biomarkers_actionability.csv: per-cancer table with gene symbol, cancer type, 'actionable' flag, and PubMed-association status
- Per-cancer actionable-gene enrichment p-values (KIRC p = 8.10 × 10−4; LIHC p = 2.23 × 10−3; PRAD not significant, 115 actionable hits)

## How to apply

Load the extracted IC gene symbol lists from the repository's Extracted folder, named 'IC_'+str(tumor)+'.csv' per cancer type (KIRC, LIHC, PRAD). Load the Supplementary data file DB_pharmacologically_active.csv downloaded from DrugBank and match gene symbols against its pharmacologically active target entries to assign an 'actionable' flag per gene; for KIRC this flags MT-CYB, NDUFV3, PARP3, and TOP1MT. Retrieve the PubMed evaluation results from the 'Knowledge-based evaluation of the results.ipynb' notebook and mark IC genes not associated with the corresponding cancer type in PubMed as novel candidate biomarkers. Compile a single per-cancer table with gene symbol, cancer type, 'actionable' flag, and PubMed-association status, saved as candidate_biomarkers_actionability.csv. Optionally test whether actionable genes are significantly enriched per cancer type (reported as p = 8.10 × 10−4 for KIRC and 2.23 × 10−3 for LIHC; PRAD is not significantly enriched despite 115 actionable hits).

## Related tools

- **Python** (Scripted matching of IC gene symbols against the DrugBank pharmacologically active target file and compilation of the actionability table) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Hosts 'Knowledge-based evaluation of the results.ipynb', which performs the DrugBank evaluation and PubMed literature association) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb

## Examples

```
python knowledge_based_evaluation.py --ic Extracted/IC_KIRC.csv --drugbank "Supplementary data/DB_pharmacologically_active.csv" --out candidate_biomarkers_actionability.csv
```

## Evaluation signals

- The output candidate_biomarkers_actionability.csv contains named candidate genes via substring check (e.g., MT-CYB and TOP1MT for KIRC).
- The output includes an 'actionable' field per gene.
- Per-cancer candidate row counts match the reported values: 13 actionable genes for LIHC and 24 for PRAD.
- KIRC actionable genes include MT-CYB, NDUFV3, PARP3, and TOP1MT, matching the DrugBank labels in the article.
- Enrichment p-values for actionable genes reproduce the reported order of magnitude (KIRC 8.10 × 10−4, LIHC 2.23 × 10−3, PRAD not significant).

## Limitations

- DrugBank version is not specified; no accession IDs or knowledge-base version details are given for DrugBank, BioGRID, or PubMed, so exact reproducibility depends on the snapshot of DB_pharmacologically_active.csv in the repository.
- PRAD actionable genes are not significantly enriched as a set, so actionability alone is not evidence of relevance for every cancer type.
- PubMed-based novelty flags rely on automated literature retrieval; genes lacking any literature association with the cancer type may reflect incomplete indexing rather than true novelty.

## Evidence

- [discussion] for KIRC disease, MT-CYB, NDUFV3, PARP3, and TOP1MT gene products are labeled as actionable in DrugBank: "for KIRC disease, MT-CYB, NDUFV3, PARP3, and TOP1MT gene products are labeled as actionable in DrugBank"
- [results] pharmacologically active targets in DrugBank: "pharmacologically active targets in DrugBank (https://www.drugbank.ca/releases/latest)"
- [readme] The jupyter notebook [Knowledge-based evaluation of the results.ipynb] ... contains systematic and statistic evaluation of the extracted genes (here called IC genes).: "The jupyter notebook [Knowledge-based evaluation of the results.ipynb](https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb) contains"
- [readme] one folder [Extracted] that contains the data regarding all the genes extracted with the pipeline: "one folder [Extracted](https://github.com/DEIB-GECO/GeneNetFusion/tree/master/Extracted) that contains the data regarding all the genes extracted with the pipeline"
