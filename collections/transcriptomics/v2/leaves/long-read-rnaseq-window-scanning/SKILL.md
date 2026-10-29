---
name: long-read-rnaseq-window-scanning
description: Use when the input is PacBio long-read RNA-seq BAM alignments and the
  goal is a genome-wide scan for candidate endogenous dsRNA regions that feed into
  novel-long-dsRNA and structured-RNA classification (e.g. reproducing dsRID Fig.
  3A counts).
when_to_use_negative:
- Input is short-read (e.g. Illumina) RNA-seq rather than PacBio long-read RNA-seq
  BAM data — skipping-read evidence and the pre-trained model were built for long
  reads from AD brain
- You want to retrain the random-forest model rather than reuse the pre-trained one
  — use the full bash dsRID.sh training pipeline instead of the extract_whole.py/model_predict.py
  prediction-only path
- The goal is variant calling or transcript isoform reconstruction from the BAM, not
  genome-wide dsRNA region prediction
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: dsRID
  role: Provides src/extract_whole.py (2500-nt window / 1250-nt slide genome scan)
    and src/model_predict.py (random-forest scoring), plus pre-trained model data
    and dsRID_novel_dsRNA.bed predictions
  repo: https://github.com/gxiaolab/dsRID
- name: PacBio
  role: Source long-read RNA-seq platform whose BAM alignments are the scanning input;
    models were trained on PacBio data derived from Alzheimer's disease (AD) brain
- name: Python 3.x
  role: Runtime for extract_whole.py and model_predict.py with packages installed
    via pip install -r requirements.txt
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:0708101a5c16fb4cb00227f7cef33a18963871317b66978daac665422e83c0ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - PacBio
  - Python
  - Python 3.x
  - dsRID
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# long-read-rnaseq-window-scanning

## Summary

Genome-wide window scanning of PacBio long-read RNA-seq alignments with dsRID's extract_whole.py followed by random-forest scoring via model_predict.py, to nominate candidate double-stranded RNA (dsRNA) regions in silico and classify them into novel long dsRNAs and structured RNAs.

## When to use

Use when the input is PacBio long-read RNA-seq BAM alignments and the goal is a genome-wide scan for candidate endogenous dsRNA regions that feed into novel-long-dsRNA and structured-RNA classification (e.g. reproducing dsRID Fig. 3A counts).

## When NOT to use

- Input is short-read (e.g. Illumina) RNA-seq rather than PacBio long-read RNA-seq BAM data — skipping-read evidence and the pre-trained model were built for long reads from AD brain
- You want to retrain the random-forest model rather than reuse the pre-trained one — use the full bash dsRID.sh training pipeline instead of the extract_whole.py/model_predict.py prediction-only path
- The goal is variant calling or transcript isoform reconstruction from the BAM, not genome-wide dsRNA region prediction

## Inputs

- PacBio long-read RNA-seq BAM alignment files (one per sample/dataset)
- Pre-trained random-forest model data (data/Pacbio_AD_data.tsv shipped with dsRID)

## Outputs

- dsRID_whole.tsv per input BAM, containing genome-scan windows with a predicted-probability column from the random-forest model
- Aggregated candidate dsRNA regions saved as data/dsRID_novel_dsRNA.bed
- Class counts: 82,266 candidate regions, 32,391 novel long dsRNAs (RNAfold-based, EER-overlapping windows removed), 49,875 structured RNAs
- data/Encode_novel_dsRNA_source.tsv with AD enriched, control enriched, and AD-control shared dsRNAs (ENCODE AD cohort)

## How to apply

For each input PacBio BAM, run src/extract_whole.py to scan the genome in 2500-nt windows with a 1250-nt slide, retaining only windows supported by >=6 reads including >=1 skipping read (skipping reads signal intron-spanning coverage characteristic of dsRNA candidates). Then run src/model_predict.py with the pre-trained model data (-i /data/Pacbio_AD_data.tsv) to assign each window a random-forest predicted probability of being a candidate dsRNA region; dsRID_whole.tsv carries this predicted-probability column. Filter candidates at predicted probability >0.5 and remove any windows overlapping EER dsRNAs to define novel long dsRNAs, saving predictions as data/dsRID_novel_dsRNA.bed. Finally apply RNAfold secondary-structure curation to classify structured RNAs. This prediction-only path opt-out of the dsRID.sh training process, which is appropriate when the pre-trained PacBio AD-brain model is reused.

## Related tools

- **dsRID** (Provides src/extract_whole.py (2500-nt window / 1250-nt slide genome scan) and src/model_predict.py (random-forest scoring), plus pre-trained model data and dsRID_novel_dsRNA.bed predictions) — https://github.com/gxiaolab/dsRID
- **PacBio** (Source long-read RNA-seq platform whose BAM alignments are the scanning input; models were trained on PacBio data derived from Alzheimer's disease (AD) brain)
- **Python 3.x** (Runtime for extract_whole.py and model_predict.py with packages installed via pip install -r requirements.txt)

## Examples

```
python src/extract_whole.py -b <input_bam_file> -o <output_dir>/dsRID_whole.tsv; python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>
```

## Evaluation signals

- Aggregating dsRID_whole.tsv outputs across all 11 datasets reproduces the reported 82,266 candidate regions, 32,391 novel long dsRNAs, and 49,875 structured RNAs, matching Fig. 3A
- Each output dsRID_whole.tsv contains a predicted-probability column from the random forest model trained on dsRID_data.tsv
- Every retained scan window satisfies the support filter: >=6 reads including >=1 skipping read
- Novel long dsRNAs have predicted probability >0.5 and show no overlap with EER dsRNAs
- Final predicted regions are written to data/dsRID_novel_dsRNA.bed in valid BED format

## Limitations

- No version of dsRID or specific package versions is pinned (requirements.txt lists packages without versions)
- The pre-trained model (Pacbio_AD_data.tsv) derives from Alzheimer's disease brain; transfer to other tissues, conditions, or platforms is not validated here
- Windows need >=6 reads including >=1 skipping read, so low-coverage regions will be missed by the genome scan
- Structured-RNA classification depends on RNAfold curation, adding a secondary-structure-prediction dependency to the counts

## Evidence

- [methods] extract_whole.py command: "python src/extract_whole.py -b <input_bam_file> -o <output_dir>/dsRID_whole.tsv"
- [methods] model_predict.py command: "python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>"
- [other] window support filter: [evidence span withheld — anchor status: not_found]
- [readme] predicted probability column: "dsRID_whole will contain column of predicted probability to be a candidate dsRNA region from the random forest model trained on dsRID_data.tsv"
- [readme] opt-out of training: "If you would like to opt out training process and directly proceed to prediction step, run"
- [results] predicted regions availability: "dsRID predicted regions across all samples are available under data/dsRID_novel_dsRNA.bed"
