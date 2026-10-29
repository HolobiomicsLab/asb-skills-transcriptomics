---
name: dsrna-region-threshold-filtering
description: Use when you need to predict dsRNA regions in silico from PacBio long-read
  RNA-seq BAM files genome-wide, and classify predicted windows into candidate dsRNAs,
  novel long dsRNAs, and structured RNAs.
when_to_use_negative:
- Input is short-read RNA-seq data rather than PacBio long-read RNA-seq BAM files
- The goal is de novo training of the dsRID model on new data rather than applying
  the pre-trained model for prediction (use the full dsRID.sh training pipeline instead)
- The input regions are already classified structured RNAs and only RNAfold curation
  is needed
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3524
tools:
- name: dsRID
  role: Machine learning-based method (extract_whole.py and model_predict.py) to scan
    long-read RNA-seq data for candidate dsRNA regions and score them with a random
    forest model
  repo: https://github.com/gxiaolab/dsRID
- name: PacBio
  role: Long-read RNA-seq technology producing the BAM input data used for dsRNA region
    discovery
- name: Python
  role: Runtime environment (Python 3.x) executing extract_whole.py and model_predict.py
    with packages installed via pip requirements.txt
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:bbb247a2134dff57f6bf0506ffc57b0095228c41552c06095a0b16ed1f5ac4df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - PacBio
  - Python
  - dsRID
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# dsrna-region-threshold-filtering

## Summary

Applies dsRID, a random-forest machine learning method, to PacBio long-read RNA-seq data to identify and classify candidate double-stranded RNA (dsRNA) regions genome-wide, filtering predictions at a probability threshold of >0.5 and removing overlaps with known EER dsRNAs to define novel long dsRNAs.

## When to use

Use when you need to predict dsRNA regions in silico from PacBio long-read RNA-seq BAM files genome-wide, and classify predicted windows into candidate dsRNAs, novel long dsRNAs, and structured RNAs.

## When NOT to use

- Input is short-read RNA-seq data rather than PacBio long-read RNA-seq BAM files
- The goal is de novo training of the dsRID model on new data rather than applying the pre-trained model for prediction (use the full dsRID.sh training pipeline instead)
- The input regions are already classified structured RNAs and only RNAfold curation is needed

## Inputs

- PacBio long-read RNA-seq data in BAM format
- Pre-trained random-forest model data (data/Pacbio_AD_data.tsv from the dsRID repository)

## Outputs

- dsRID_whole.tsv containing per-window predicted probabilities of being a candidate dsRNA region
- data/dsRID_novel_dsRNA.bed with genomic coordinates of predicted novel dsRNA regions
- Counts of candidate dsRNA regions, novel long dsRNAs, and structured RNAs (82,266 / 32,391 / 49,875 across 11 datasets)

## How to apply

Run src/extract_whole.py on each PacBio BAM file to scan the genome in 2500-nt windows with a 1250-nt slide, retaining only windows supported by >=6 reads including >=1 skipping read, producing dsRID_whole.tsv. Run src/model_predict.py with the pre-trained model data (-i /data/Pacbio_AD_data.tsv) to assign each window a random-forest predicted probability of being a candidate dsRNA region. Filter candidates at predicted probability >0.5 and remove windows overlapping EER dsRNAs to define novel long dsRNAs. Aggregate outputs across all 11 datasets to reproduce the reported 82,266 candidate regions and 32,391 novel long dsRNAs, saving predictions as data/dsRID_novel_dsRNA.bed. Apply RNAfold secondary-structure curation to classify structured RNAs, reproducing the reported 49,875 structured RNAs.

## Related tools

- **dsRID** (Machine learning-based method (extract_whole.py and model_predict.py) to scan long-read RNA-seq data for candidate dsRNA regions and score them with a random forest model) — https://github.com/gxiaolab/dsRID
- **PacBio** (Long-read RNA-seq technology producing the BAM input data used for dsRNA region discovery)
- **Python** (Runtime environment (Python 3.x) executing extract_whole.py and model_predict.py with packages installed via pip requirements.txt)

## Examples

```
python src/extract_whole.py -b <input_bam_file> -o <output_dir>/dsRID_whole.tsv; python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>
```

## Evaluation signals

- dsRID_whole.tsv must contain a column of predicted probability to be a candidate dsRNA region from the random forest model trained on dsRID_data.tsv
- Aggregated candidate-region, novel-long-dsRNA, and structured-RNA counts (82,266 / 32,391 / 49,875) must match the values reported in Fig. 3A of the paper
- Predicted novel dsRNA regions must be written to data/dsRID_novel_dsRNA.bed as in the published repository
- Retained windows from extract_whole.py must satisfy the >=6 supporting reads threshold including >=1 skipping read
- model_predict.py must additionally produce feat_importance and cv_scores files (feature importance and cross-validation accuracy results)

## Limitations

- No version of dsRID or specific package versions stated; required packages are only listed in the requirements.txt file
- Prediction step depends on pre-trained model data (Pacbio_AD_data.tsv) from Alzheimer's disease brain, so transfer to other tissues or conditions is unvalidated
- The structured-RNA classification step relies on RNAfold secondary-structure curation of candidate regions, which adds a downstream dependency

## Evidence

- [intro] dsRID (Double-Stranded RNA Identifier) is a machine learning-based method designed to predict dsRNA regions in silico: "dsRID (Double-Stranded RNA Identifier) is a machine learning-based method designed to predict dsRNA regions in silico by leveraging the power of long-read RNA sequencing"
- [methods] python src/extract_whole.py -b <input_bam_file> -o <output_dir>/dsRID_whole.tsv: "python src/extract_whole.py -b <input_bam_file> -o <output_dir>/dsRID_whole.tsv"
- [readme] dsRID_whole will contain column of predicted probability to be a candidate dsRNA region from the random forest model: "dsRID_whole will contain column of predicted probability to be a candidate dsRNA region from the random forest model trained on dsRID_data.tsv"
- [results] we applied dsRID to an AD cohort sequenced by the ENCODE consortium: "We applied dsRID to an AD cohort sequenced by the ENCODE consortium and characterized the global dsRNA profile with potentially distinct expression patterns between AD and controls"
- [results] 32,391 were categorized as novel long dsRNAs based on RNAfold, and 49,875 were denoted as structured RNAs: [evidence span withheld — anchor status: not_found]
