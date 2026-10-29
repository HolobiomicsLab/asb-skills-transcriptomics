---
name: go-enrichment-analysis-of-gene-lists
description: Use when you have gene lists flagged as critical by a drug-response model
  (e.g., scDEAL integrated-gradient gene output) and need functional interpretation
  via GO term enrichment with adjusted p-values.
when_to_use_negative:
- Input is a ranked genome-wide gene list rather than a focused set of model-derived
  critical genes — the skill interprets IG-selected signature genes, not arbitrary
  gene panels
- The single-cell model has not yet been run with --printgene 'T', so no critical
  gene lists exist to enrich
- The dataset is not matched to a bulk drug-screening training set (e.g., GDSC/CCLE-derived)
  — scDEAL's accuracy varies depending on the collection of bulk gene expression of
  cell lines
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: scDEAL
  role: Generates the critical gene lists via integrated gradient interpretation (--printgene
    'T') that are the input to GO enrichment
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: PyTorch
  role: Backend for training the bulk and single-cell DaNN models whose IG scores
    define the gene lists
- name: Scanpy
  role: Package applied to the AnnData (.h5ad) object holding single-cell predictions
    and sens_label annotations
- name: AnnData
  role: Output format storing single-cell predictions (adata.obs['sens_label']) used
    alongside the critical gene lists
- name: conda-pack
  role: Distributes the pinned scDEAL conda environment (scdeal.tar.gz) needed to
    reproduce the enrichment run
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:55f410ce0bb03da2887ed3c8a0164e87f4d672254c0fbe285be6048d9f4ba024
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - PyTorch
  - Scanpy
  - conda-pack
  - scDEAL
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# go-enrichment-analysis-of-gene-lists

## Summary

Performs GO enrichment analysis on drug-response critical gene lists derived from scDEAL's integrated gradient (IG) scores — e.g., 936 sensitive and 868 resistant genes from the Cisplatin-treated OSCC dataset GSE117872 — to recover biological terms like 'DNA repair' and 'cell division'. It converts model-interpretability output into biologically interpretable annotations.

## When to use

Use when you have gene lists flagged as critical by a drug-response model (e.g., scDEAL integrated-gradient gene output) and need functional interpretation via GO term enrichment with adjusted p-values.

## When NOT to use

- Input is a ranked genome-wide gene list rather than a focused set of model-derived critical genes — the skill interprets IG-selected signature genes, not arbitrary gene panels
- The single-cell model has not yet been run with --printgene 'T', so no critical gene lists exist to enrich
- The dataset is not matched to a bulk drug-screening training set (e.g., GDSC/CCLE-derived) — scDEAL's accuracy varies depending on the collection of bulk gene expression of cell lines

## Inputs

- Critical gene lists from scDEAL integrated gradient output (drug-sensitive and drug-resistant gene sets, e.g., 936 sensitive and 868 resistant genes)
- Preprocessed GSE117872 scRNA-seq data and bulk drug-screening files (ALL_expression.csv, ALL_label_binary_wf.csv)
- casestudy/analysis_criticalgenes.ipynb notebook template

## Outputs

- GO enriched terms with adjusted p-values for the sensitive-cell critical gene list (e.g., 'DNA repair' adj. p = 0.039)
- GO enriched terms with adjusted p-values for the resistant-cell critical gene list (e.g., 'cell division' adj. p = 0.003)
- Single-cell predictions stored as AnnData .h5ad with adata.obs['sens_label'] (0 = resistance, 1 = sensitivity), e.g., GSE117872[timestamp].h5ad

## How to apply

First generate the critical gene lists by running scDEAL's bulkmodel.py and scmodel.py with the integrated gradient flag (--printgene 'T'), keeping encoder_h_dims/bulk_h_dims (256,128), predictor_h_dims (128,64), and bottleneck identical across both steps. For the GSE117872 (HN120, Cisplatin) case, use the DAE encoder, dropout 0.3, lr 0.01, and SMOTE sampling as suggested in the README parameter table; this yields lists of drug-sensitive and drug-resistant genes with significantly differential IG scores (936 and 868 genes respectively). Then run the GO enrichment on these two lists using the casestudy/analysis_criticalgenes.ipynb notebook. Judge correctness by whether the enrichment reproduces reported terms — 'DNA repair' (adj. p = 0.039) and 'cell division' (adj. p = 0.003) — under expert review.

## Related tools

- **scDEAL** (Generates the critical gene lists via integrated gradient interpretation (--printgene 'T') that are the input to GO enrichment) — https://github.com/OSU-BMBL/scDEAL.git
- **PyTorch** (Backend for training the bulk and single-cell DaNN models whose IG scores define the gene lists)
- **Scanpy** (Package applied to the AnnData (.h5ad) object holding single-cell predictions and sens_label annotations)
- **AnnData** (Output format storing single-cell predictions (adata.obs['sens_label']) used alongside the critical gene lists)
- **conda-pack** (Distributes the pinned scDEAL conda environment (scdeal.tar.gz) needed to reproduce the enrichment run)

## Examples

```
python bulkmodel.py --drug "CISPLATIN" --dimreduce "DAE" --encoder_h_dims "256,128" --predictor_h_dims "128,64" --bottleneck 512 --data_name "GSE117872" --sampling "SMOTE" --dropout 0.3 --lr 0.01 --printgene "T" --checkpoint "False"
```

## Evaluation signals

- AnnData output .h5ad exists under save/adata/data/ with a sens_label field in adata.obs (0 = resistance, 1 = sensitivity)
- Sensitive-group enrichment recovers 'DNA repair' with adj. p = 0.039
- Resistant-group enrichment recovers 'cell division' with adj. p = 0.003
- Critical gene counts match the reported scale (936 drug-sensitive genes in HN120P; 868 drug-resistant genes in HN120PCR), all with significantly differential IG scores
- Encoder/predictor dimensions were identical across bulkmodel.py and scmodel.py runs (--encoder_h_dims/bulk_h_dims '256,128', --predictor_h_dims '128,64', bottleneck 512), otherwise IG gene lists are invalid

## Limitations

- Prediction accuracy could vary depending on the collection of bulk gene expression of cell lines; scDEAL training data may change as additional bulk-level databases are integrated
- A potential sample swap between Data 1 and 2 (HN120 and HN137) was pointed out by a recent study, which affects interpretation of these critical gene lists
- No software version numbers are stated for tools or the environment, so exact reproducibility of enrichment p-values may vary

## Evidence

- [abstract] Integrated gradient feature interpretation: "Another feature of scDEAL is the integrated gradient feature interpretation to infer the signature genes of drug resistance mechanisms."
- [intro] IG interpretability rationale: [evidence span withheld — anchor status: not_found]
- [readme] Identical dimensions across bulk and single-cell steps: "Remember that the dimension of the encoder and predictor should be identical (--encoder_h_dims(bulk_h_dims) "256,128", --predictor_h_dims "128,64", --bottleneck 256) in two steps."
- [readme] GSE117872 Cisplatin suggested parameters: "GSE117872HN120 | CISPLATIN | 512 | 256,128 | 128,64 | DAE | 0.3 | 0.01 | SMOTE"
- [other] GO enrichment reproduction and validation thresholds: [evidence span withheld — anchor status: not_found]
