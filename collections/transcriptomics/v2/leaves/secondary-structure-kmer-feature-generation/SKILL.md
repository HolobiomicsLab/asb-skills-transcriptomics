---
name: secondary-structure-kmer-feature-generation
description: Use when training or predicting with LncDC with secondary structure features
  enabled (the '-r' flag), so RNAfold foldings must be converted into ss k-mer score
  tables at k=1..k=5.
when_to_use_negative:
- If secondary structure features are not wanted, train without '-r' — output is then
  only the hexamer table and SIF+PF model files (XGB_model_SIF_PF.pkl)
- If ViennaRNA is not installed and cannot be installed, since RNAfold is required
  to compute the foldings the ss k-mer tables depend on
- If the input is already a computed feature matrix — this skill only generates ss
  k-mer tables from transcript FASTA sequences
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: LncDC
  role: lncDC-train.py with the '-r' flag folds transcripts and generates the five
    ss k-mer tables; lncDC.py consumes them via '-r -k' for prediction
  repo: https://github.com/lim74/LncDC
- name: ViennaRNA
  role: RNAfold performs the secondary structure calculation on which the SASS k-mer
    features are based; verified by 'import RNA'
  repo: https://www.tbi.univie.ac.at/RNA/
- name: XGBoost
  role: receives the SASS (SSF) features together with SIFs and PFs in the trained
    SIF_PF_SSF model
  repo: https://github.com/lim74/LncDC
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:7ca21512c21c829eaaeb3da45920173f913e01d4dcc44c0590da20b3f3da4dbd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lim74/LncDC
  tools:
  - COME
  - CPAT
  - CPC2
  - CPPred
  - LncDC
  - LncFinder
  - PLEK
  - Python
  - ViennaRNA
  - XGBoost
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.631578947368421
n_claims: 19
---

# secondary-structure-kmer-feature-generation

## Summary

Generates sequence-and-secondary-structure (SASS) k-mer score features for RNA transcripts by folding each transcript with RNAfold (ViennaRNA) and scoring k-mers (k=1 to 5) against training-derived ss k-mer tables, which improves LncDC's lncRNA vs mRNA classification.

## When to use

Use when training or predicting with LncDC with secondary structure features enabled (the '-r' flag), so RNAfold foldings must be converted into ss k-mer score tables at k=1..k=5.

## When NOT to use

- If secondary structure features are not wanted, train without '-r' — output is then only the hexamer table and SIF+PF model files (XGB_model_SIF_PF.pkl)
- If ViennaRNA is not installed and cannot be installed, since RNAfold is required to compute the foldings the ss k-mer tables depend on
- If the input is already a computed feature matrix — this skill only generates ss k-mer tables from transcript FASTA sequences

## Inputs

- mRNA FASTA file (-m), e.g. mrna_mouse.fasta, plain or gzip-compressed
- matching CDS FASTA file (-c) with the same order and number of records as the mRNA file
- lncRNA FASTA file (-l), e.g. lncrna_mouse.fasta

## Outputs

- Five ss k-mer tables: <prefix>_ss_table_k1.csv through <prefix>_ss_table_k5.csv (e.g. SS_mouse_ss_table_k1.csv ... k5.csv)
- Trained SIF+PF+SSF model artifacts with -r: <prefix>_xgb_model_SIF_PF_SSF.pkl, <prefix>_imputer_SIF_PF_SSF.pkl, <prefix>_scaler_SIF_PF_SSF.pkl
- Prediction CSV (via lncDC.py -r -k) labeling transcripts as 'lncrna' or 'mrna' using SS features

## How to apply

Install ViennaRNA (e.g. 'conda install -c bioconda viennarna') and verify the Python binding with 'import RNA'. Train with the '-r' flag on lncDC-train.py (e.g. '-m mrna_mouse.fasta -c cds_mouse.fasta -l lncrna_mouse.fasta -o SS_mouse -r -t 8'); this folds every transcript with RNAfold and emits five ss k-mer tables named <prefix>_ss_table_k1.csv through <prefix>_ss_table_k5.csv. For prediction, supply both '-r' and '-k <ss_table_prefix>' to lncDC.py so the transcript foldings are scored against the SS tables and combined with SIF and PF features in the XGBoost model (XGB_model_SIF_PF_SSF.pkl). Expect longer runtimes because secondary structure calculation by RNAfold is time-consuming; use the '-t' thread parameter to parallelize. Rationale: SASS k-mer scores integrate primary sequence with secondary structure information, and the paper reports these features improved classification capability.

## Related tools

- **LncDC** (lncDC-train.py with the '-r' flag folds transcripts and generates the five ss k-mer tables; lncDC.py consumes them via '-r -k' for prediction) — https://github.com/lim74/LncDC
- **ViennaRNA** (RNAfold performs the secondary structure calculation on which the SASS k-mer features are based; verified by 'import RNA') — https://www.tbi.univie.ac.at/RNA/
- **XGBoost** (receives the SASS (SSF) features together with SIFs and PFs in the trained SIF_PF_SSF model) — https://github.com/lim74/LncDC

## Examples

```
python bin/lncDC-train.py -m mrna_mouse.fasta -c cds_mouse.fasta -l lncrna_mouse.fasta -o SS_mouse -r -t 8
```

## Evaluation signals

- Training with '-r' must emit exactly five ss tables, named <prefix>_ss_table_k1.csv through <prefix>_ss_table_k5.csv
- Verification that ViennaRNA is functional: 'python -c "import RNA"' completes with no error messages
- Prediction with SS features must include both '-r' and '-k <prefix>' arguments matching the trained table prefix (e.g. 'mouse_ss_table')
- Resulting model files must carry the SSF suffix (e.g. XGB_model_SIF_PF_SSF.pkl), not the SIF_PF suffix
- LncDC models including SSFs should reach or exceed benchmark accuracy (mouse model ~0.9758 accuracy / 0.9516 MCC on M-Test)

## Limitations

- Secondary structure prediction with RNAfold is time-consuming for large transcript sets
- The paper notes the accuracy gain from SSF selection is marginal because SIFs and PFs already achieve high accuracy, making further improvement difficult
- For non-model organisms with too few annotated lncRNAs, ss tables must be borrowed from evolutionary close model organisms (e.g. Zebrafish, Drosophila melanogaster)

## Evidence

- [abstract] SASS k-mer score features improve classification: "the use of sequence and secondary structure (SASS) k-mer score features and flexible ORF features improved the classification capability of LncDC"
- [readme] training with SS features generates five ss tables: "five ss tables will also be generated, including SS_mouse_ss_table_k1.csv, SS_mouse_ss_table_k2.csv, SS_mouse_ss_table_k3.csv, SS_mouse_ss_table_k4.csv and SS_mouse_ss_table_k5.csv"
- [readme] ViennaRNA required for SS features: "ViennaRNA (Optional, only required when using secondary structure features for prediction)"
- [readme] this may take a longer time because secondary structure calculation by RNAfold is time-consuming: "This may take a longer time because secondary structure calculation by RNAfold (The main program of ViennaRNA) is time-consuming."
