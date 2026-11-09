---
name: coding-potential-tool-benchmarking
description: Use when you have curated mRNA (with matched CDS) and lncRNA transcript
  FASTA sets split into train/test, and need to reproduce or extend a Table 2-style
  comparison of LncDC against six existing coding-potential tools.
when_to_use_negative:
- You only need a single tool's predictions on one transcript set with no cross-tool
  comparison — a plain LncDC prediction run suffices.
- The input is already a numeric feature matrix or a metrics table rather than transcript
  FASTA sequences — the benchmarking skill consumes raw FASTA through each tool's
  own feature extraction.
- Sequences are not lncRNA/mRNA coding-potential candidates (e.g., protein FASTA or
  genomic contigs) — the six tools and LncDC expect RNA transcript FASTA.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: LncDC
  role: Primary XGBoost-based lncRNA detector being benchmarked; trained with lncDC-train.py
    and applied with lncDC.py, optionally with SSFs via RNAfold
  repo: https://github.com/lim74/LncDC
- name: XGBoost
  role: Classifier selected after LR/DT/SVM/RF comparison; produces the LncDC model
    pickle used for test-set predictions
- name: ViennaRNA
  role: RNAfold computes secondary structures for the SASS k-mer ss tables during
    '-r' training/prediction
- name: CPAT
  role: Baseline coding-potential tool run on the same test set for comparison
- name: PLEK
  role: Baseline coding-potential tool run on the same test set for comparison
- name: CPC2
  role: Baseline coding-potential tool run on the same test set for comparison
- name: CPPred
  role: Baseline coding-potential tool run on the same test set for comparison
- name: LncFinder
  role: Baseline coding-potential tool run on the same test set for comparison
- name: COME
  role: Baseline coding-potential tool run on the same test set for comparison
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:b7856befe0d3515410d8c79a91bb8ef6d917ebf32b7b051c43b9aba787fc6723
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
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# coding-potential-tool-benchmarking

## Summary

Benchmark lncRNA/mRNA coding-potential classifiers by running LncDC (self-trained mouse/human XGBoost models, with or without SASS k-mer secondary structure features) alongside CPAT, PLEK, CPC2, CPPred, LncFinder, and COME on the same held-out test FASTA sets, then compute and compare accuracy, precision, specificity, and MCC.

## When to use

Use when you have curated mRNA (with matched CDS) and lncRNA transcript FASTA sets split into train/test, and need to reproduce or extend a Table 2-style comparison of LncDC against six existing coding-potential tools.

## When NOT to use

- You only need a single tool's predictions on one transcript set with no cross-tool comparison — a plain LncDC prediction run suffices.
- The input is already a numeric feature matrix or a metrics table rather than transcript FASTA sequences — the benchmarking skill consumes raw FASTA through each tool's own feature extraction.
- Sequences are not lncRNA/mRNA coding-potential candidates (e.g., protein FASTA or genomic contigs) — the six tools and LncDC expect RNA transcript FASTA.

## Inputs

- mRNA transcript FASTA for training (e.g., mrna_mouse.fasta)
- CDS FASTA in matching order/number to the mRNA FASTA (e.g., cds_mouse.fasta)
- lncRNA transcript FASTA for training (e.g., lncrna_mouse.fasta)
- Held-out test transcript FASTA (e.g., M-Test or H-Test)

## Outputs

- Hexamer table CSV (e.g., self_mouse_hexamer_table.csv)
- XGBoost model pickle (e.g., self_mouse_xgb_model_SIF_PF.pkl) plus imputer and scaler pickles
- Secondary structure k-mer tables k1–k5 (when trained with '-r')
- Per-transcript lncrna/mrna prediction CSVs for LncDC and each baseline tool
- Benchmark comparison table of accuracy, precision, specificity, and MCC per tool (e.g., table2_mouse_benchmark.csv)

## How to apply

Assemble the train/test transcript sets (e.g., M-Train/M-Test mouse FASTAs: mRNA transcripts, their CDS in matching order, and lncRNA transcripts), then train the LncDC model with 'python bin/lncDC-train.py -m mrna_mouse.fasta -c cds_mouse.fasta -l lncrna_mouse.fasta -o self_mouse -t 8', which internally applies RFECV feature selection and SMOTE balancing and emits the hexamer table, XGBoost model, imputer, and scaler. Re-train with '-r' to add SASS k-mer secondary structure features, which invokes RNAfold (ViennaRNA) and generates the five ss k-mer tables k1–k5. Predict on the test set with 'python bin/lncDC.py' using the self-trained model (pointing '-x', '-m', '-p', '-s' at the emitted files, and '-r -k' at the ss table prefix when using SSFs), writing lncrna/mrna labels to CSV. Run CPAT, PLEK, CPC2, CPPred, LncFinder, and COME on the identical test FASTA, then compute accuracy, precision, specificity, and MCC for every tool and assemble the comparison table (e.g., table2_mouse_benchmark.csv).

## Related tools

- **LncDC** (Primary XGBoost-based lncRNA detector being benchmarked; trained with lncDC-train.py and applied with lncDC.py, optionally with SSFs via RNAfold) — https://github.com/lim74/LncDC
- **XGBoost** (Classifier selected after LR/DT/SVM/RF comparison; produces the LncDC model pickle used for test-set predictions)
- **ViennaRNA** (RNAfold computes secondary structures for the SASS k-mer ss tables during '-r' training/prediction)
- **CPAT** (Baseline coding-potential tool run on the same test set for comparison)
- **PLEK** (Baseline coding-potential tool run on the same test set for comparison)
- **CPC2** (Baseline coding-potential tool run on the same test set for comparison)
- **CPPred** (Baseline coding-potential tool run on the same test set for comparison)
- **LncFinder** (Baseline coding-potential tool run on the same test set for comparison)
- **COME** (Baseline coding-potential tool run on the same test set for comparison)

## Examples

```
python bin/lncDC.py -i test/mouse_test_toy.fasta -o lncdc_ss_mouse.csv -x data/train_hexamer_table.csv -m data/XGB_model_SIF_PF_SSF.pkl -p data/imputer_SIF_PF_SSF.pkl -s data/scaler_SIF_PF_SSF.pkl -r -k data/train_ss_table -t 8
```

## Evaluation signals

- Expected LncDC mouse model results on M-Test: accuracy ≈0.9758 and MCC ≈0.9516 (human model ≈0.9723/0.9446), ahead of LncFinder (0.9620/0.9240) and CPAT (0.9615/0.9230).
- The mRNA FASTA and CDS FASTA must contain the same number of sequences in the same order, as required by lncDC-train.py's '-c' input.
- Training with '-r' must produce exactly five ss k-mer table files (k1–k5) with the chosen output prefix, e.g., SS_mouse_ss_table_k1.csv … k5.csv.
- Each tool's prediction output uses consistent lncrna/mrna labels on the identical test FASTA so accuracy, precision, specificity, and MCC are computed over the same transcript set.
- The resulting comparison table should show LncDC outperforming all six baselines, reproducing the Table 2 ordering.

## Limitations

- Secondary structure feature extraction via RNAfold is time-consuming, so '-r' runs (training and prediction) take substantially longer.
- For non-model organisms lacking well-annotated lncRNAs, training may be unreliable; the README suggests borrowing models from evolutionarily close organisms (Zebrafish, Yeast, Soybean, Drosophila melanogaster, Caenorhabditis elegans, Arabidopsis thaliana, Oryza sativa).
- Specific package versions are required (pandas 1.5, numpy 1.23, scikit-learn 1.1.3, xgboost 1.7.1, imbalanced-learn 0.9.1, biopython >=1.79, tqdm >=4.64); verify with 'python test_requirements.py'.

## Evidence

- [full_text] On M-Test, the LncDC mouse model achieved the best performance among all evaluated tools, with an accuracy of 0.9758 and an MCC score of 0.9516, outperforming LncFinder (0.9620/0.9240) and CPAT (0.9615/0.9230).: [evidence span withheld — anchor status: not_found]
- [abstract] Benchmarking scope: "Benchmarking experiments showed that LncDC consistently outperformed six state-of-the-art tools in distinguishing lncRNAs from mRNAs"
- [readme] Training invocation: "python bin/lncDC-train.py -m mrna_mouse.fasta -c cds_mouse.fasta -l lncrna_mouse.fasta -o self_mouse -t 8"
- [readme] SSF training output: "five ss tables will also be generated, including SS_mouse_ss_table_k1.csv, SS_mouse_ss_table_k2.csv, SS_mouse_ss_table_k3.csv, SS_mouse_ss_table_k4.csv and SS_mouse_ss_table_k5.csv."
- [discussion] XGBoost selection rationale: "Five popular algorithms, LR, DT, SVM, RF, and XGBoost were compared in this study. Because XGBoost outperformed other algorithms in performance evaluation, it was used in the LncDC program"
- [readme] CDS ordering constraint: "The order and number of the CDS sequences should be the same as the mRNA sequences."
- [readme] Runtime caveat: "This may take a longer time because secondary structure calculation by RNAfold (The main program of ViennaRNA) is time-consuming."
