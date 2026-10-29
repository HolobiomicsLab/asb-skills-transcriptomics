---
name: rna-transcript-dataset-construction
description: Use when building training and test FASTA sets of mRNA, their CDS, and
  lncRNA transcripts from curated annotations (e.g., NCBI RefSeq or GENCODE) to train
  or benchmark lncRNA identification models.
when_to_use_negative:
- Raw RNA-Seq reads rather than assembled transcript FASTA — LncDC's predictor takes
  RNA transcript sequences in FASTA format, not FASTQ read data
- Feature tables or precomputed k-mer matrices — this skill constructs the raw transcript-level
  FASTA inputs that precede feature extraction
- Organisms with too few well-annotated lncRNAs to build a local training set — the
  README recommends instead reusing models trained on evolutionary close model organisms
  (Zebrafish, Yeast, Soybean, D. melanogaster, etc.)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0659
tools:
- name: LncDC
  role: Consumes the assembled mRNA/CDS/lncRNA FASTA sets via lncDC-train.py to build
    the hexamer table, XGBoost model, imputer, and scaler, and via lncDC.py to predict
    labels on the test FASTA
  repo: https://github.com/lim74/LncDC
- name: ViennaRNA
  role: Invoked as RNAfold during '-r' training to generate the SASS k-mer secondary
    structure tables (k1–k5) from the transcript FASTA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:bd1b2f1b0379785d687d81e6d5b9f582608f978874ac5a8ea934668bf2c83755
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# rna-transcript-dataset-construction

## Summary

Assembles curated mRNA, CDS, and lncRNA transcript FASTA sets (e.g., M-Train/M-Test mouse sets) from manual gene annotations so they can be fed to LncDC's lncDC-train.py and lncDC.py for model training and coding-potential benchmarking.

## When to use

Use when building training and test FASTA sets of mRNA, their CDS, and lncRNA transcripts from curated annotations (e.g., NCBI RefSeq or GENCODE) to train or benchmark lncRNA identification models.

## When NOT to use

- Raw RNA-Seq reads rather than assembled transcript FASTA — LncDC's predictor takes RNA transcript sequences in FASTA format, not FASTQ read data
- Feature tables or precomputed k-mer matrices — this skill constructs the raw transcript-level FASTA inputs that precede feature extraction
- Organisms with too few well-annotated lncRNAs to build a local training set — the README recommends instead reusing models trained on evolutionary close model organisms (Zebrafish, Yeast, Soybean, D. melanogaster, etc.)

## Inputs

- Curated mouse (or human) gene annotations, e.g., NCBI RefSeq manually curated annotations or GENCODE
- mRNA transcript FASTA file (e.g., mrna_mouse.fasta)
- CDS FASTA file in matching order and count to the mRNA transcripts (e.g., cds_mouse.fasta)
- lncRNA transcript FASTA file (e.g., lncrna_mouse.fasta)

## Outputs

- M-Train and M-Test FASTA partitions of mRNA, CDS, and lncRNA sequences
- Hexamer table CSV, XGBoost model pickle, imputer pickle, and scaler pickle with a chosen output prefix (e.g., self_mouse_*)
- Optionally five secondary structure k-mer tables (e.g., SS_mouse_ss_table_k1.csv through _k5.csv) when '-r' is used
- Prediction CSV on the test set with 'lncrna'/'mrna' labels (e.g., lncdc_mouse.csv)
- Benchmark table (e.g., table2_mouse_benchmark.csv) with accuracy, precision, specificity, and MCC per tool

## How to apply

Extract from curated mouse annotations the mRNA transcript sequences (mrna_mouse.fasta), the CDS sequences in matching order (cds_mouse.fasta), and the lncRNA transcripts (lncrna_mouse.fasta), splitting them into an M-Train set for model training and an M-Test set for evaluation. The CDS file must preserve the order and number of the mRNA transcripts, because LncDC pairs each transcript with its CDS to compute flexible ORF features capturing CDS characteristics beyond the longest ORF. Train with 'python bin/lncDC-train.py -m mrna_mouse.fasta -c cds_mouse.fasta -l lncrna_mouse.fasta -o self_mouse -t 8', which produces a hexamer table, XGBoost model pickle, imputer, and scaler under the given prefix (RFECV feature selection and SMOTE balancing are applied internally); retrain with '-r' to add SASS k-mer secondary structure tables k1–k5 via RNAfold/ViennaRNA. Predict on the held-out M-Test FASTA with 'python bin/lncDC.py' using the self-trained model files (-x/-m/-p/-s, plus -r -k with the ss-table prefix for SSF mode), and run baselines (CPAT, PLEK, CPC2, CPPred, LncFinder, COME) on the identical test set for a fair Table 2 comparison. Correctness is judged by class balance across train/test, matched CDS/transcript counts, and LncDC mouse model reaching accuracy ≈0.9758 and MCC ≈0.9516 on M-Test.

## Related tools

- **LncDC** (Consumes the assembled mRNA/CDS/lncRNA FASTA sets via lncDC-train.py to build the hexamer table, XGBoost model, imputer, and scaler, and via lncDC.py to predict labels on the test FASTA) — https://github.com/lim74/LncDC
- **ViennaRNA** (Invoked as RNAfold during '-r' training to generate the SASS k-mer secondary structure tables (k1–k5) from the transcript FASTA)

## Examples

```
python bin/lncDC-train.py -m mrna_mouse.fasta -c cds_mouse.fasta -l lncrna_mouse.fasta -o self_mouse -t 8
```

## Evaluation signals

- The CDS FASTA contains the same number of sequences as the mRNA FASTA and in the identical order, as required by lncDC-train.py
- Training outputs all four expected files under the chosen prefix: hexamer table CSV, XGB model pickle, imputer pickle, and scaler pickle (plus five ss_table k-mer CSVs when '-r' is used)
- Prediction output CSV assigns only the two labels 'lncrna' and 'mrna' to every test transcript
- The self-trained mouse model achieves accuracy ≈0.9758 and MCC ≈0.9516 on M-Test (human model ≈0.9723/0.9446), outperforming LncFinder (0.9620/0.9240) and CPAT (0.9615/0.9230)
- 'python test_requirements.py' passes in the conda environment (python ≥3.9, scikit-learn 1.1.3, xgboost 1.7.1, imbalanced-learn 0.9.1) before training

## Limitations

- Secondary structure feature training and prediction (-r) is markedly slower because RNAfold structure calculation is time-consuming
- The CDS file must exactly match the mRNA file in order and number, otherwise transcript–CDS pairing breaks; RefSeq itself contains 1909 mRNA transcripts whose CDS differs from the longest ORF, so naive ORF-based dataset construction loses these cases
- For non-model organisms with few annotated lncRNAs, self-training on local data may not be feasible; transfer from close model organisms is the README's suggested workaround

## Evidence

- [readme] The mRNA sequences are stored in the mrna_mouse.fasta file, and their corresponding CDS sequences are stored in the cds_mouse.fasta file.: "The mRNA sequences are stored in the mrna_mouse.fasta file, and their corresponding CDS sequences are stored in the cds_mouse.fasta file."
- [readme] The order and number of the CDS sequences should be the same as the mRNA sequences.: "The order and number of the CDS sequences should be the same as the mRNA sequences."
- [intro] in the gold-standard NCBI RefSeq manually curated gene annotations, there are 1909 mRNA transcripts whose coding sequence (CDS) differ from the longest ORF: "in the gold-standard NCBI RefSeq manually curated gene annotations, there are 1909 mRNA transcripts whose coding sequence (CDS) differ from the longest ORF"
- [intro] lncFinder randomly selects 8,000 lncRNAs and 8,000 mRNAs from the GENCODE database to construct the training dataset of human: "lncFinder randomly selects 8,000 lncRNAs and 8,000 mRNAs from the GENCODE database to construct the training dataset of human"
