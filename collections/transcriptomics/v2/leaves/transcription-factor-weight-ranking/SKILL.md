---
name: transcription-factor-weight-ranking
description: Use when TFvelo has been run on a scRNA-seq dataset (rc.h5ad with TF
  weight matrix) and you need to identify the top-weighted regulators or verify TF
  weight signs on pathway genes.
when_to_use_negative:
- The TFvelo run has not been executed or rc.h5ad with the TF weight matrix is unavailable
- The goal is pathway/GO over-representation of best-fitted genes rather than TF regulator
  ranking (use enrichment analysis instead)
- The dataset lacks identifiable transcription factors in the ENCODE/ChEA candidate
  TF databases
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: TFvelo
  role: Fits the TF-based gene dynamics model whose weight matrix is ranked; weights
    initialized by correlation and optimized with lsq_linear using ENCODE+ChEA candidate
    TFs
  repo: github.com/xiaoyeye/TFvelo
- name: anndata
  role: Reads the rc.h5ad output containing the TF weight matrix
- name: gseapy
  role: Provides pathway gene sets (e.g., KEGG insulin secretion) used to restrict
    the weight matrix before sign-consistency checks
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:86eae715492a2cd090480874b9931456b6e1f01e96928b734bf7e7fd0275a801
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TFvelo
  - anndata
  - gseapy
  - scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# transcription-factor-weight-ranking

## Summary

Rank transcription factors by their normalized TFvelo model weights on best-fitted genes to identify key regulators of differentiation, optionally checking weight-sign consistency on pathway-restricted gene sets.

## When to use

Use when TFvelo has been run on a scRNA-seq dataset (rc.h5ad with TF weight matrix) and you need to identify the top-weighted regulators or verify TF weight signs on pathway genes.

## When NOT to use

- The TFvelo run has not been executed or rc.h5ad with the TF weight matrix is unavailable
- The goal is pathway/GO over-representation of best-fitted genes rather than TF regulator ranking (use enrichment analysis instead)
- The dataset lacks identifiable transcription factors in the ENCODE/ChEA candidate TF databases

## Inputs

- rc.h5ad file from TFvelo run output folder containing best-fitted genes and the TF weight matrix
- Optional gene-set restriction list (e.g., KEGG insulin secretion pathway genes)

## Outputs

- Ranked list of transcription factors by normalized absolute model weight
- Verification table of TF weight signs on pathway genes (e.g., written to 'KEGG_enrichment_results.csv')

## How to apply

Load the rc.h5ad file from the dataset-specific TFvelo output folder (e.g., 'TFvelo_pancreas/rc.h5ad' produced by 'python TFvelo_run_demo.py --dataset_name pancreas'), extracting the best-fitted genes and their TF weight matrix produced with default hyperparameters (init_weight_method=correlation, WX_method=lsq_linear, TF_databases=ENCODE+ChEA). Restrict the weight matrix to a gene set of interest if applicable (e.g., insulin secretion pathway genes), normalize the TF weights, and rank TFs by normalized absolute weight. Count TFs whose absolute normalized weight exceeds 0.5 as high-absolute-weight regulators. Optionally verify weight-sign consistency: e.g., REST should be consistently negative and HMGN3 consistently positive on insulin secretion pathway genes; on gastrulation erythroid data, the top ranking should recover GATA1, GATA2 and LMO2 consistent with Fig. 4g, h.

## Related tools

- **TFvelo** (Fits the TF-based gene dynamics model whose weight matrix is ranked; weights initialized by correlation and optimized with lsq_linear using ENCODE+ChEA candidate TFs) — github.com/xiaoyeye/TFvelo
- **anndata** (Reads the rc.h5ad output containing the TF weight matrix)
- **gseapy** (Provides pathway gene sets (e.g., KEGG insulin secretion) used to restrict the weight matrix before sign-consistency checks)

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas  # then load TFvelo_pancreas/rc.h5ad, normalize TF weights, rank TFs by |weight| > 0.5
```

## Evaluation signals

- On pancreas data, REST exhibits consistently negative weights and HMGN3 consistently positive weights on insulin secretion pathway genes
- REST and HMGN3 exceed the normalized |weight| > 0.5 threshold and appear among high-absolute-weight regulators
- On gastrulation erythroid data, the top-weight TF ranking recovers GATA1, GATA2 and LMO2 consistent with Fig. 4g, h
- Ranking is computed only over best-fitted genes from the converged TFvelo model (rc.h5ad exists and contains the weight matrix)

## Limitations

- The current linear model for estimating TF weights may be insufficient; the article notes 'more complex models may be explored to replace the current linear model in TFvelo for estimating the weight of each TFs'
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14), so weights for poorly fitted genes may be unreliable
- TFvelo is slower than scVelo, which could be a weakness for large-scale datasets

## Evidence

- [readme] Candidate TF selection and weight init: "--init_weight_method: The method to initialize the weights. Correlation is adopted by default."
- [readme] Weight optimization method: "--WX_method: The method to optimize weight. lsq_linear is adopted by default."
- [methods] TF databases: "--TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default."
- [discussion] Key TF detection claim: "TFvelo can be used to infer the pseudo time, cell trajectory and detect key TFs which play important role in the differentiation"
- [discussion] Linear model limitation: "more complex models may be explored to replace the current linear model in TFvelo for estimating the weight of each TFs"
