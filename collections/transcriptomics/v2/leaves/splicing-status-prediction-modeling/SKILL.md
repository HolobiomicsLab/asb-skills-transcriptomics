---
name: splicing-status-prediction-modeling
description: Use when you need per-read splicing probabilities for single-cell RNA-seq
  alignments—especially reads labeled ambiguous ('A') by alevin-fry/STAR workflow
  logic—so that spliced/unspliced quantification or downstream velocity analyses can
  resolve fundamentally ambiguous reads.
when_to_use_negative:
- Reads already have definitive splicing labels from paired-end alignment rules (exon
  containment patterns e∧e′, e·e′, i, i·e on read 1) — no prediction is needed
- The task is gene-level or spliced/unspliced counting only and ambiguous reads can
  simply be discarded rather than probabilistically resolved
- Input is bulk RNA-seq or non-10X single-end data without oligo(dT) priming information,
  since the model is trained on scRNA-seq priming windows
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Forseti
  role: Python package providing the forseti_predictor master function that combines
    the trained binding-affinity MLP and fragment-length spline to assign splicing
    probabilities
  repo: https://github.com/COMBINE-lab/forseti
- name: STAR
  role: Aligns reads to the spliceu index with --runMode genomeGenerate (--sjdbOverhang
    100), --soloType CB_UMI_Simple, --quantMode TranscriptomeSAM, --outSAMtype BAM
    SortedByCoordinate
- name: samtools
  role: Filters uniquely genome-mapped reads with view -d NH:1 and pulls matching
    toTranscriptome alignments with view -N
- name: bedtools
  role: Identifies fully exonic (ambiguous) reads via intersect -f 1.0 -wo -bed against
    exon_by_tx.bed
- name: R
  role: Runs build_spliceu.R to construct the spliceu augmented transcriptome reference
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:cd673ef584695d659c73b4267b5785760700ce2af656aa46756f782147f7609b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - Python
  - R
  - STAR
  - bedtools
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# splicing-status-prediction-modeling

## Summary

Forseti-style mechanistic modeling that probabilistically assigns a spliced/unspliced status to scRNA-seq reads by combining an MLP-trained oligo(dT) binding-affinity model with a spline-fit fragment length distribution, scoring putative fragments against proximate priming sites.

## When to use

Use when you need per-read splicing probabilities for single-cell RNA-seq alignments—especially reads labeled ambiguous ('A') by alevin-fry/STAR workflow logic—so that spliced/unspliced quantification or downstream velocity analyses can resolve fundamentally ambiguous reads.

## When NOT to use

- Reads already have definitive splicing labels from paired-end alignment rules (exon containment patterns e∧e′, e·e′, i, i·e on read 1) — no prediction is needed
- The task is gene-level or spliced/unspliced counting only and ambiguous reads can simply be discarded rather than probabilistically resolved
- Input is bulk RNA-seq or non-10X single-end data without oligo(dT) priming information, since the model is trained on scRNA-seq priming windows

## Inputs

- 10X-style paired-end scRNA-seq FASTQ files (e.g. pbmc_1k_v3 R1/R2 gzipped FASTQs with 12 bp UMI, 3M-february-2018.txt whitelist)
- Reference genome FASTA and gene annotation GTF (e.g. refdata-gex-GRCh38-2020-A)
- Genome-uniquely-mapped toTranscriptome BAM from STAR (--quantMode TranscriptomeSAM)
- List of uniquely mapped read names (from samtools view -d NH:1)
- Trained model checkpoints (mlp_pa6with1mm.pkl binding-affinity MLP and spline fragment-length model)

## Outputs

- Per-read splicing status probabilities (spliced vs unspliced) for ambiguous reads
- spliceu augmented reference files: spliceu.gtf, spliceu.fa, t2g_3col.tsv
- Trained binding-affinity checkpoint mlp_pa6with1mm.pkl and fitted fragment-length spline
- Evaluation metrics: ROC curves and AUCs (e.g. forseti_eval_auc_results.csv)

## How to apply

First build the spliceu augmented transcriptome (spliced + unspliced transcript versions) with build_spliceu.R from a reference GTF and FASTA, producing spliceu.gtf, spliceu.fa, and t2g_3col.tsv. Align reads with STAR using --soloType CB_UMI_Simple, --quantMode TranscriptomeSAM, --outSAMtype BAM SortedByCoordinate, then filter genome-uniquely-mapped reads via samtools view -d NH:1 and use bedtools intersect with -f 1.0 -wo against exon_by_tx.bed to find fully exonic (ambiguous) reads. Train the binding-affinity MLP on labeled priming windows containing an adenine single-nucleotide repeat (aSNR) with snr_min_size=6 ('pa6with1mm' configuration), reserving a 1000-example per-dataset hold-out, and fit the fragment length distribution with a cubic spline; persist the model as mlp_pa6with1mm.pkl. Finally call forseti_predictor on the toTranscriptome BAM, unimap read names, spliceu.fa, and t2g_3col.tsv with defaults (snr_min_size=6, discount_perc=1, polya_tail_len=200, max_frag_len=1000) to obtain per-read spliced/unspliced probabilities.

## Related tools

- **Forseti** (Python package providing the forseti_predictor master function that combines the trained binding-affinity MLP and fragment-length spline to assign splicing probabilities) — https://github.com/COMBINE-lab/forseti
- **STAR** (Aligns reads to the spliceu index with --runMode genomeGenerate (--sjdbOverhang 100), --soloType CB_UMI_Simple, --quantMode TranscriptomeSAM, --outSAMtype BAM SortedByCoordinate)
- **samtools** (Filters uniquely genome-mapped reads with view -d NH:1 and pulls matching toTranscriptome alignments with view -N)
- **bedtools** (Identifies fully exonic (ambiguous) reads via intersect -f 1.0 -wo -bed against exon_by_tx.bed)
- **R** (Runs build_spliceu.R to construct the spliceu augmented transcriptome reference)

## Examples

```
STAR --runThreadN 12 --genomeDir $STAR_IDX_DIR --readFilesCommand zcat --outSAMtype BAM SortedByCoordinate --quantMode TranscriptomeSAM --soloType CB_UMI_Simple --soloUMIlen 12 --soloBarcodeReadLength 0
```

## Evaluation signals

- Trained MLP reproduces mean accuracies of ~0.83 on training windows, ~0.84 on per-dataset 1000-example hold-out sets, and ~0.89 on the two held-out test datasets
- Prediction AUCs on experimental evaluation sets match reported values of 0.89 (mouse) and 0.85 (human), mean 0.88; mean AUC 0.92 on simulated data
- spliceu build outputs spliceu.gtf, spliceu.fa, and t2g_3col.tsv exist and STAR genomeGenerate completes with --sjdbOverhang 100
- Every input read in the toTranscriptome BAM that is scored appears in the unimap read-name list (NH:1 filtering verified)
- Ambiguous reads ('A'), i.e. reads compatible with both spliced and unspliced transcripts, all receive a predicted splicing probability from the model

## Limitations

- As a mechanistic model, Forseti cannot distinguish fundamentally ambiguous reads: reads from polyA tail priming whose read2 lies in the terminal exon versus reads whose original cDNA fragment arises entirely from an exon
- Model parameters are tuned to 10X-style scRNA-seq (snr_min_size=6, discount_perc=1, polya_tail_len=200, max_frag_len=1000); other library protocols may not transfer
- Large datasets are computationally heavy — the PBMC1k tutorial extracts the top 200 cells (cells with ≥3000 genes) to cut the full 200-million-alignment run down to 65,784,347 alignments (~3.3 hours on 32 threads)
- Running the full forseti-experiments Nextflow pipeline requires ~3 TB of disk space and a SLURM-compatible cluster configuration

## Evidence

- [abstract] Forseti obtains a mean Area Under the Receiver Operating Characteristic Curve (AUC) score of 0.92 on simulated data, and 0.88 on the experimental data.: "Forseti obtains a mean Area Under the Receiver Operating Characteristic Curve (AUC) score of 0.92 on simulated data, and 0.88 on the experimental data."
- [readme] first, we train a binding affinity model to assign a probability that a given transcriptomic site is used in fragment generation: "first, we train a binding affinity model to assign a probability that a given transcriptomic site is used in fragment generation"
- [methods] The R script `build_spliceu.R` will build a _spliceu_: "call the master function named `forseti_predictor` for the splicing status prediction"
- [methods] Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts; the prediction model processes these ambiguous reads and predicts splicing status for them: "Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts. Our prediction model processes these ambiguous reads and accurately predict splicing status"
