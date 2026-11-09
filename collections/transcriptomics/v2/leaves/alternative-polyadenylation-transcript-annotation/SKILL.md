---
name: alternative-polyadenylation-transcript-annotation
description: Use when RNA-Seq alignments and a reconstructed transcriptome (GTF) are
  available and you need to identify expressed polyA sites or refine 3'-terminal exons
  without 3'-end sequencing data.
when_to_use_negative:
- You need de novo detection of coding alternative polyadenylation in upstream exons
  — aptardi only evaluates 3' terminal exons, so upstream APA cannot be identified.
- You require exact single-nucleotide polyA site resolution — aptardi predicts on
  100-base bins, though sites wiggle up to ~30 nucleotides.
- Your input transcriptome has many isoforms sharing 3' terminal exons and you cannot
  apply the RSEM-based filtering — aptardi will enumerate redundant aptardi transcripts
  for each shared isoform.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: aptardi
  role: Predicts expressed polyA sites and refines transcript 3'-ends by combining
    RNA-Seq coverage with DNA sequence using a pre-built machine learning model
  repo: https://github.com/luskry/aptardi
- name: StringTie
  role: Reconstructs the input transcriptome (stringtie.gtf) whose 3' terminal exons
    aptardi evaluates
- name: BEDtools
  role: Intersects predicted positive bins with PolyASite 2.0 / PolyA_DB 3 annotations
    to classify true and false positives
- name: RSEM
  role: Quantitates the aptardi transcriptome before and after filtering to remove
    transcripts without at least 1 count in 2/3 of samples
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:4336e8c8c6340dba7c4283aedf4d15d72f973910f7f4e3c862868f7c42a72ee5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - BEDtools
  - RSEM
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# alternative-polyadenylation-transcript-annotation

## Summary

Predict sample-specific polyadenylation (polyA) sites and refine transcript 3'-ends by running aptardi, a supervised machine learning tool that combines RNA-Seq read coverage with genome DNA sequence, and evaluate the predicted bins against polyA-site reference databases (e.g., PolyASite 2.0, PolyA_DB 3) with BEDtools.

## When to use

Use when RNA-Seq alignments and a reconstructed transcriptome (GTF) are available and you need to identify expressed polyA sites or refine 3'-terminal exons without 3'-end sequencing data.

## When NOT to use

- You need de novo detection of coding alternative polyadenylation in upstream exons — aptardi only evaluates 3' terminal exons, so upstream APA cannot be identified.
- You require exact single-nucleotide polyA site resolution — aptardi predicts on 100-base bins, though sites wiggle up to ~30 nucleotides.
- Your input transcriptome has many isoforms sharing 3' terminal exons and you cannot apply the RSEM-based filtering — aptardi will enumerate redundant aptardi transcripts for each shared isoform.

## Inputs

- Sorted BAM of genome-aligned RNA-Seq reads (e.g., HISAT2-aligned HBR sample)
- Genome FASTA (e.g., hg38.fa)
- Input transcriptome GTF/GFF (designed for StringTie output)
- Pre-built model file (model.hdf5) and scale file (scale.pk) for Mode 1
- PolyA-site reference annotations (PolyASite 2.0, PolyA_DB 3) for evaluation

## Outputs

- Modified transcriptome GTF/GFF containing original plus aptardi transcripts with annotated 3'-ends (annotated as the 3'-most base of predicted positive 100-base bins)
- Predicted positive polyA-site bins (probability >= 0.5) as BED-format intervals
- Evaluation metrics table (precision, recall, analyzed/true-positive/false-positive bin counts per reference database)

## How to apply

Align reads with HISAT2 (e.g., --rna-strandness RF --dta), sort the BAM with SAMtools, and reconstruct the transcriptome with StringTie. Run aptardi in Mode 1 with the pre-built HBR model (model.hdf5 and scale.pk from src/aptardi/ml_scale) using default parameters: 100-base prediction bins (--w 100), probability threshold 0.5 (--p 0.5), maximum 300 analyzed bins per transcript (--i 300); aptardi only evaluates 3' terminal exons of input transcripts and outputs a GTF with aptardi transcripts added alongside the originals. Extract predicted positive bins (probability >= 0.5) as BED intervals keyed to analyzed 3' terminal exons, then intersect them with PolyASite 2.0 or PolyA_DB 3 annotations using BEDtools to classify true positives and false positives and compute precision/recall per database. For multi-sample projects, follow the recommended filtering pipeline: quantitate the aptardi transcriptome with RSEM, remove transcripts without at least 1 count in 2/3 of samples, then re-quantitate with RSEM for downstream differential analysis. Mode 2 (--m) with gold-standard polyA sites (--s with 0-based chromosome/strand/site column positions via --l) should only be used to train a custom model, since the pre-built model requires 100-base bins.

## Related tools

- **aptardi** (Predicts expressed polyA sites and refines transcript 3'-ends by combining RNA-Seq coverage with DNA sequence using a pre-built machine learning model) — https://github.com/luskry/aptardi
- **StringTie** (Reconstructs the input transcriptome (stringtie.gtf) whose 3' terminal exons aptardi evaluates)
- **BEDtools** (Intersects predicted positive bins with PolyASite 2.0 / PolyA_DB 3 annotations to classify true and false positives)
- **RSEM** (Quantitates the aptardi transcriptome before and after filtering to remove transcripts without at least 1 count in 2/3 of samples)

## Examples

```
aptardi --o outdir --f hg38.fa --r stringtie.gtf --b sorted.bam --n model.hdf5 --t scale.pk --g aptardi.gtf -p 0.5 -w 100
```

## Evaluation signals

- Reproduced precision and recall against each reference database match reported values within small tolerance (PolyASite 2.0: precision 0.85, recall 0.18; PolyA_DB 3: precision 0.88, recall 0.22).
- Per-database bin counts are reproduced (e.g., 569,005 and 289,998 annotated sites yielding 102,774 and 86,685 analyzed bins).
- Output GTF contains original input transcripts plus added aptardi transcripts — aptardi must not remove any input transcripts.
- All predicted positive bins have probability >= 0.5 and stop sites are annotated as the 3'-most base of 100-base bins.
- Filtering metrics are consistent: removed transcripts are exactly those with <1 RSEM count in 2/3 of samples.

## Limitations

- Prediction resolution is limited to 100-base bins; polyA sites can wiggle up to ~30 nucleotides and few bins contain multiple sites.
- aptardi does not distinguish transcripts overlapping the same genomic region, so a single 3'-end may be incorporated into multiple transcripts per gene, and a stop site may be attributed to the wrong transcript of the same gene.
- The pre-built HBR model should not be used with altered bin sizes (--w), and manually engineered DNA sequence features may not apply outside mammals.

## Evidence

- [readme] aptardi evaluates only 3' terminal exons: "Note that aptardi does not evaluate intron junctions but rather only evalutes 3' terminal exons of input transcripts and annotates 3' ends accordingly."
- [readme] probability threshold parameter: "--p/-p <float>			Probability threshold, predictions >= threshold are labeled transription stop site (default: 0.5, value must be constrained by (0, 1))"
- [discussion] 100-base bin resolution: "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
- [methods] core method description: "combines RNA-Seq data from the sample with the genome (DNA sequence) of the sample to identify 3' ends of transcripts using machine learning"
- [methods] filtering rationale: "a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
