---
name: knowledge-base-integration
description: Use when a gene list (e.g., IC genes from the Extracted folder as 'IC_'+str(tumor)+'.csv')
  must be cross-referenced against DrugBank pharmacologically active targets and PubMed
  to flag actionable genes and novel candidate biomarkers per cancer type.
when_to_use_negative:
- Input is raw TCGA gene expression data rather than an already-extracted IC gene
  list — run Pipeline.ipynb and community extraction with Gephi first
- The goal is computational validation via normal/cancer classification performance
  (AUC/accuracy/F1) rather than knowledge-based actionability/literature annotation
- No knowledge-base context is needed and the genes being annotated are not from the
  fused-network IC gene sets of this paper
tools:
- name: Jupyter Notebook
  role: Runs the 'Knowledge-based evaluation of the results.ipynb' notebook that performs
    the DrugBank actionability matching and PubMed literature evaluation of IC genes
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
- name: Python
  role: Implements the gene-symbol matching against DB_pharmacologically_active.csv
    and compilation of candidate_biomarkers_actionability.csv
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: DrugBank
  role: Source of pharmacologically active target annotations used to flag IC genes
    as drug actionable
- name: PubMed
  role: Source of automatic literature retrieval used to mark IC genes without cancer-type
    association as novel candidate biomarkers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:19f2389866de478ebd40c08ee17cbe7d3ee2df4f97ff8739eaf938e9ca40bddd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DrugBank
  - Jupyter Notebook
  - PubMed
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# knowledge-base-integration

## Summary

Annotates extracted IC genes from the fused KIRC, LIHC, and PRAD co-expression networks with drug-actionability flags from DrugBank pharmacologically active targets and PubMed literature association, yielding per-cancer candidate biomarker tables that separate known from novel candidates.

## When to use

Use when a gene list (e.g., IC genes from the Extracted folder as 'IC_'+str(tumor)+'.csv') must be cross-referenced against DrugBank pharmacologically active targets and PubMed to flag actionable genes and novel candidate biomarkers per cancer type.

## When NOT to use

- Input is raw TCGA gene expression data rather than an already-extracted IC gene list — run Pipeline.ipynb and community extraction with Gephi first
- The goal is computational validation via normal/cancer classification performance (AUC/accuracy/F1) rather than knowledge-based actionability/literature annotation
- No knowledge-base context is needed and the genes being annotated are not from the fused-network IC gene sets of this paper

## Inputs

- IC gene symbol lists per cancer type from the Extracted folder ('IC_KIRC.csv', 'IC_LIHC.csv', 'IC_PRAD.csv')
- DB_pharmacologically_active.csv downloaded from DrugBank (Supplementary data folder)
- PubMed evaluation results from 'Knowledge-based evaluation of the results.ipynb'

## Outputs

- candidate_biomarkers_actionability.csv with gene symbol, cancer type, 'actionable' flag, and PubMed-association status
- Per-cancer actionable gene counts and enrichment p-values (KIRC 8.10 × 10−4, LIHC 2.23 × 10−3, PRAD not significant)
- List of novel candidate biomarkers (IC genes with no PubMed association to their cancer type)

## How to apply

Load the IC gene symbol lists for KIRC, LIHC, and PRAD from the repository's Extracted folder (files named 'IC_'+str(tumor)+'.csv'), plus the DB_pharmacologically_active.csv file from the Supplementary data folder. Match gene symbols against DrugBank's pharmacologically active target entries to assign an 'actionable' flag per gene (e.g., MT-CYB, NDUFV3, PARP3, and TOP1MT for KIRC). Retrieve the PubMed evaluation results from the 'Knowledge-based evaluation of the results.ipynb' notebook and mark IC genes lacking any literature association with the corresponding cancer type as novel candidate biomarkers. Compile a single per-cancer table (gene symbol, cancer type, 'actionable' flag, PubMed-association status) saved as candidate_biomarkers_actionability.csv, and run enrichment testing of actionable genes per cancer type — expected significance for KIRC (p = 8.10 × 10−4) and LIHC (p = 2.23 × 10−3), with PRAD not significantly enriched despite 115 actionable hits.

## Related tools

- **Jupyter Notebook** (Runs the 'Knowledge-based evaluation of the results.ipynb' notebook that performs the DrugBank actionability matching and PubMed literature evaluation of IC genes) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
- **Python** (Implements the gene-symbol matching against DB_pharmacologically_active.csv and compilation of candidate_biomarkers_actionability.csv) — https://github.com/DEIB-GECO/GeneNetFusion
- **DrugBank** (Source of pharmacologically active target annotations used to flag IC genes as drug actionable)
- **PubMed** (Source of automatic literature retrieval used to mark IC genes without cancer-type association as novel candidate biomarkers)

## Examples

```
pip install -r requirements.txt; jupyter notebook "Knowledge-based evaluation of the results.ipynb"
```

## Evaluation signals

- candidate_biomarkers_actionability.csv must contain the named candidate genes by substring check (e.g., MT-CYB and TOP1MT for KIRC)
- Output must include an 'actionable' field per gene
- Per-cancer actionable candidate row counts must match reported values: 13 actionable genes for LIHC and 24 for PRAD
- Enrichment p-values must reproduce KIRC 8.10 × 10−4 and LIHC 2.23 × 10−3, with PRAD not significantly enriched but containing 115 actionable hits
- PRAD actionable hits must include the 4 targets of PRAD-treatment drugs: ACPP, MAP1A, VKORC1, VKORC1L1

## Limitations

- No accession IDs or data availability details are given for the knowledge-base versions used (DrugBank, BioGRID, PubMed), so exact reproduction depends on the release snapshot stored in the repository
- PRAD actionable genes are not significantly enriched, so actionability alone is weaker evidence for that cancer type
- The README's step-by-step run description focuses on the LIHC analysis; Gephi community extraction is manual, so regenerating the IC gene lists upstream is not fully automated

## Evidence

- [results] DrugBank pharmacologically active targets: "pharmacologically active targets in DrugBank (https://www.drugbank.ca/releases/latest)"
- [discussion] KIRC actionable genes in DrugBank: "for KIRC disease, MT-CYB, NDUFV3, PARP3, and TOP1MT gene products are labeled as actionable in DrugBank"
- [discussion] Novel biomarker definition via PubMed: "IC genes that may be novel cancer biomarkers are those not associated with the cancer type in PubMed"
- [readme] DrugBank supplementary file in repository: [evidence span withheld — anchor status: not_found]
- [readme] Notebook for knowledge-based evaluation: [evidence span withheld — anchor status: not_found]
