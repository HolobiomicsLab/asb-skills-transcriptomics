---
name: paired-end-splicing-label-derivation
description: Use when building an evaluation read set with definitive splicing ground
  truth from paired-end 10X scRNA-seq data (e.g., mouse/human scRNA-seq) to benchmark
  splicing-status predictors via AUC.
when_to_use_negative:
- 'Single-end libraries: the labeling rules depend on read 1''s exon-containment pattern
  to label the read-2 mate, so a paired-end alignment is required.'
- 'The goal is prediction rather than ground-truth construction: for probabilistic
  splicing-status prediction run the Forseti model (forseti_predictor) instead of
  deriving labels.'
- Reads that do not map uniquely to the genome (multiple NH values) — the labeling
  pipeline explicitly filters them out, so they cannot receive definitive labels.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: STAR
  role: Aligns paired-end reads to the spliceu index and emits transcriptomic alignments
    (toTranscriptome BAM) used for exon-containment analysis
  repo: https://github.com/COMBINE-lab/forseti
- name: samtools
  role: Filters genome-uniquely-mapped reads (-d NH:1) and pulls matching toTranscriptome
    alignments by read name (-N)
  repo: https://github.com/COMBINE-lab/forseti
- name: bedtools
  role: Intersects alignments with exon_by_tx.bed (-f 1.0 -wo -bed) to find reads
    100% mapped in an exonic region
  repo: https://github.com/COMBINE-lab/forseti
- name: Forseti
  role: Predictor whose per-read splicing probabilities are scored against the derived
    ground-truth labels to compute ROC/AUC
  repo: https://github.com/COMBINE-lab/forseti
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:9ab20ae81df88e40e6da325081ad14e957f1cd102993dc77dbde127a230cf3fb
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# paired-end-splicing-label-derivation

## Summary

Derives definitive ground-truth spliced/unspliced labels for scRNA-seq reads by exploiting paired-end alignment structure: the exon-containment pattern of read 1 determines the splicing status assigned to its read-2 mate. This creates an evaluation read set whose labels are treated as ground truth for computing ROC/AUC against Forseti's probabilistic predictions.

## When to use

Use when building an evaluation read set with definitive splicing ground truth from paired-end 10X scRNA-seq data (e.g., mouse/human scRNA-seq) to benchmark splicing-status predictors via AUC.

## When NOT to use

- Single-end libraries: the labeling rules depend on read 1's exon-containment pattern to label the read-2 mate, so a paired-end alignment is required.
- The goal is prediction rather than ground-truth construction: for probabilistic splicing-status prediction run the Forseti model (forseti_predictor) instead of deriving labels.
- Reads that do not map uniquely to the genome (multiple NH values) — the labeling pipeline explicitly filters them out, so they cannot receive definitive labels.

## Inputs

- Paired-end 10X scRNA-seq FASTQ files (R1 barcode/UMI read, R2 transcript read)
- Spliceu augmented transcriptome reference (spliceu.gtf, spliceu.fa, t2g_3col.tsv)
- STAR genome-aligned BAM sorted by coordinate (Aligned.sortedByCoord.out.bam)
- STAR toTranscriptome BAM (--quantMode TranscriptomeSAM output)
- exon_by_tx.bed exon-containment annotation per transcript

## Outputs

- Evaluation read set with definitive per-read spliced/unspliced ground-truth labels assigned to read-2 mates
- ROC curves and AUC values comparing predictor probabilities against the paired-end labels (e.g., forseti_eval_auc_results.csv)

## How to apply

Align reads to the spliceu augmented transcriptome with STAR (--quantMode TranscriptomeSAM produces alignments translated into transcript coordinates). Filter to genome-uniquely-mapped reads using `samtools view -d NH:1`, extract the unimap read names with `cut -f1`, and pull the matching toTranscriptome alignments with `samtools view -N`. Use `bedtools intersect` against exon_by_tx.bed with `-f 1.0 -wo -bed` to identify reads fully contained in an exonic region (the ambiguous 'A' reads compatible with both spliced and unspliced transcripts). Then apply the Section 2.7 paired-end labeling rules: read-1 exon containment patterns (e∧e′, e·e′, i, i·e) are used to assign a definitive spliced or unspliced label to the read-2 mate, forming the evaluation read set. Compare predictor probabilities against these labels to compute ROC curves and AUCs (the paper reports 0.89 on mouse and 0.85 on human experimental sets, mean 0.88).

## Related tools

- **STAR** (Aligns paired-end reads to the spliceu index and emits transcriptomic alignments (toTranscriptome BAM) used for exon-containment analysis) — https://github.com/COMBINE-lab/forseti
- **samtools** (Filters genome-uniquely-mapped reads (-d NH:1) and pulls matching toTranscriptome alignments by read name (-N)) — https://github.com/COMBINE-lab/forseti
- **bedtools** (Intersects alignments with exon_by_tx.bed (-f 1.0 -wo -bed) to find reads 100% mapped in an exonic region) — https://github.com/COMBINE-lab/forseti
- **Forseti** (Predictor whose per-read splicing probabilities are scored against the derived ground-truth labels to compute ROC/AUC) — https://github.com/COMBINE-lab/forseti

## Examples

```
samtools view -d NH:1 Aligned.sortedByCoord.out.bam | cut -f1 > unimap_names.txt && bedtools intersect -a filtered_Aligned.sortedByCoord.out.bam -b exon_by_tx.bed -f 1.0 -wo -bed > fully_exonic.bed
```

## Evaluation signals

- All reads retained in the evaluation set pass the NH:1 filter, i.e., map uniquely to the genome.
- Fully exonic reads are identified with bedtools intersect using -f 1.0 (100% overlap with an exon_by_tx.bed feature).
- Every read-2 mate in the evaluation set carries a categorical spliced/unspliced label determined solely by its read-1 exon containment pattern (e∧e′, e·e′, i, i·e).
- AUC computed against these labels on mouse and human experimental sets falls near the reported 0.89 and 0.85 (mean 0.88).

## Limitations

- Fundamentally ambiguous reads — e.g., polyA-tail-primed fragments whose read 2 lies in the terminal exon, or fragments arising wholly from an exon — cannot be distinguished mechanistically, capping achievable AUC below 1.0.
- Labels are only assigned where read 1 yields a definitive containment pattern; ambiguous reads ('A') compatible with both spliced and unspliced transcripts are the ones left for the prediction model to classify.
- The experimental evaluation was done on datasets randomly selected from 10 processed paired-end scRNA-seq experiments (mouse and human), so label generality to other chemistries/species is not established here.

## Evidence

- [methods] we use samtools to filter the reads have unique map to the genome: "we use samtools to filter the reads have unique map to the genome"
- [methods] bedtools intersect -a filtered_Aligned.sortedByCoord.out.bam (against exon_by_tx.bed with -f 1.0): "bedtools intersect \
-a $STAR_OUT_DIR/filtered_Aligned.sortedByCoord.out.bam"
- [methods] Ambiguous reads 'A' are compatible with both spliced and unspliced transcripts: "Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts."
- [readme] STAR TranscriptomeSAM mode outputs alignments translated into transcript coordinates: "With `--quantMode TranscriptomeSAM`, STAR will output alignments translated into transcript coordinates"
- [readme] -d NH:1 flag yields uniquely mapped reads: "# `-d NH:1` with this flag, we get the uniquely mapped reads"
