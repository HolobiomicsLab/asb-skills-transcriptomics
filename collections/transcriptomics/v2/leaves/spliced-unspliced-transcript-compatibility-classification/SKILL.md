---
name: spliced-unspliced-transcript-compatibility-classification
description: Use when scRNA-seq reads aligned to a spliceu transcriptome need a probabilistic
  spliced/unspliced/ambiguous status, or when multi-gene mapped reads must be resolved
  to a single gene of origin.
when_to_use_negative:
- When the goal is standard gene/UMI quantification only — STAR --soloFeatures GeneFull
  already provides exon and intron UMI counts without splicing-status modeling.
- When reads are not aligned to a spliceu (spliced+unspliced augmented) reference;
  the model scores transcriptomic sites from the spliceu index and cannot run on a
  plain genome BAM alone.
- When distinguishing reads whose read2 lies in the terminal exon from polyA-tail-primed
  fragments — Forseti, as a mechanistic model, cannot separate these fundamentally
  ambiguous sources.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Forseti
  role: Predictive model that scores putative fragments to assign splicing status
    and identify gene origin of multi-gene mapped reads
  repo: https://github.com/COMBINE-lab/forseti
- name: STAR
  role: Aligns reads to the spliceu index producing sorted BAM with transcriptome
    alignments (--quantMode TranscriptomeSAM, --soloFeatures GeneFull)
- name: samtools
  role: Filters reads that uniquely map to the genome via the NH:1 tag
- name: bedtools
  role: bedtools intersect identifies reads 100% mapped in an exonic region (ambiguous
    reads)
- name: conda
  role: Manages the build_spliceu_r and forseti_test environments from the provided
    yml files
  repo: https://github.com/COMBINE-lab/forseti
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:bd64660461bfa0912982d0e180fd05c93b9ec791cf0e9e87c4068b5e6d84153d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - Python
  - STAR
  - bedtools
  - conda
  - samtools
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# spliced-unspliced-transcript-compatibility-classification

## Summary

Classify scRNA-seq reads by splicing status (spliced, unspliced, or ambiguous) using Forseti, a mechanistic model that combines a trained oligo(dT) binding affinity MLP with a cubic-spline fragment length distribution to score putative fragments against a spliceu (spliced+unspliced) augmented transcriptome.

## When to use

Use when scRNA-seq reads aligned to a spliceu transcriptome need a probabilistic spliced/unspliced/ambiguous status, or when multi-gene mapped reads must be resolved to a single best gene of origin.

## When NOT to use

- When the goal is standard gene/UMI quantification only — STAR --soloFeatures GeneFull already provides exon and intron UMI counts without splicing-status modeling.
- When reads are not aligned to a spliceu (spliced+unspliced augmented) reference; the model scores transcriptomic sites from the spliceu index and cannot run on a plain genome BAM alone.
- When distinguishing reads whose read2 lies in the terminal exon from polyA-tail-primed fragments — Forseti, as a mechanistic model, cannot separate these fundamentally ambiguous sources.

## Inputs

- Spliceu augmented reference (spliceu.fa, spliceu.gtf, t2g_3col.tsv) built by build_spliceu.R
- Transcriptome-aligned BAM from STAR (Aligned.toTranscriptome.out.bam)
- unimap_read_names.txt list of uniquely genome-mapped reads (NH:1)
- Pre-trained models: mlp_pa6with1mm.pkl (binding affinity MLP) and spline_model.pkl (fragment length distribution)
- 10x Genomics FASTQ files with 3M-february-2018.txt CB whitelist (e.g. PBMC1k v3 chemistry)

## Outputs

- forseti_dict mapping each read to its predicted splicing status (spliced/unspliced/ambiguous 'A')
- predictions_log with read_count, final_multi_gene_count, rescued_multigene_count, and final_ambiguous_count
- Multi-gene assignment metrics (e.g. multigene_assignment_metrics.csv with 65%/93% assignment and 75%/69% correctness targets)

## How to apply

First build the spliceu augmented reference with build_spliceu.R from a genome FASTA and GTF (producing spliceu.fa, spliceu.gtf, and t2g_3col.tsv), then align reads with STAR using --quantMode TranscriptomeSAM and --soloFeatures GeneFull so alignments are available in transcript coordinates. Filter to reads that uniquely map to the genome with samtools (keeping NH:1 tags), and identify ambiguous reads (100% exonic, compatible with both spliced and unspliced transcripts) via bedtools intersect. Activate the forseti_test conda environment and call forseti_predictor from the forseti Python package with default parameters (snr_min_size=6, discount_perc=1, polya_tail_len=200, max_frag_len=1000), passing the transcriptome BAM, unimap_read_names.txt of uniquely mapped reads, and the pre-trained spline_model.pkl and mlp_pa6with1mm.pkl models; the returned forseti_dict gives per-read splicing predictions. Reads with tied highest scores across genes are treated as multi-mapped/ambiguous, and applying the maximum-score criterion can rescue a single best gene of origin (65% and 93% of multi-gene reads in the two experimental evaluation datasets, with 75% and 69% conditional correctness).

## Related tools

- **Forseti** (Predictive model that scores putative fragments to assign splicing status and identify gene origin of multi-gene mapped reads) — https://github.com/COMBINE-lab/forseti
- **STAR** (Aligns reads to the spliceu index producing sorted BAM with transcriptome alignments (--quantMode TranscriptomeSAM, --soloFeatures GeneFull))
- **samtools** (Filters reads that uniquely map to the genome via the NH:1 tag)
- **bedtools** (bedtools intersect identifies reads 100% mapped in an exonic region (ambiguous reads))
- **conda** (Manages the build_spliceu_r and forseti_test environments from the provided yml files) — https://github.com/COMBINE-lab/forseti

## Examples

```
conda activate forseti_test; forseti_dict, predictions_log = forseti_predictor(spliceu_fa='spliceu.fa', t2g='t2g_3col.tsv', bam='Aligned.toTranscriptome.out.bam', unimap_read_names='unimap_read_names.txt', mlp_model='mlp_pa6with1mm.pkl', spline_model='spline_model.pkl', snr_min_size=6, discount_perc=1, polya_tail_len=200, max_frag_len=1000, num_threads=12)
```

## Evaluation signals

- predictions_log.read_count should match the number of uniquely genome-mapped reads fed to forseti_predictor
- final_multi_gene_count + rescued_multigene_count + final_ambiguous_count should reconcile against total multi-mapped reads, with rescued reads having a unique maximum gene score
- Forseti should achieve AUC ≈ 0.92 on simulated data and ≈ 0.88 on experimental data for splicing-status prediction
- Multi-gene assignment rates should reproduce ~65% and ~93% (with 75% and 69% conditional correctness) on the two experimental evaluation datasets
- Spliceu build sanity: spliceu.fa, spliceu.gtf, and t2g_3col.tsv all present and STAR genomeGenerate over the index completes without scaffold errors

## Limitations

- As a mechanistic model, Forseti cannot distinguish fundamentally ambiguous reads (polyA tail priming with read2 in the terminal exon vs. fragments arising from an internal exon).
- Reads whose highest gene scores are tied remain unresolvable (final_ambiguous_count); only 65%–93% of multi-gene reads could be assigned in the evaluation datasets.
- Prediction is compute-intensive: the filtered PBMC1k tutorial BAM (65,784,347 alignments after top-200-cell extraction) takes ~3.3 hours on 32 threads.
- Full forseti-experiments pipeline requires ~3 TB of disk space and a SLURM-configured Nextflow setup.

## Evidence

- [readme] Forseti combines these two trained models to predict the splicing status of the molecule of origin of reads by scoring putative fragments that associate each alignment of sequenced reads with proximate potential priming sites.: "Forseti combines these two trained models to predict the splicing status of the molecule of origin of reads by scoring putative fragments that associate each alignment of sequenced reads with"
- [methods] Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts.: "Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts. Our prediction model processes these ambiguous reads and accurately predict splicing status"
- [readme] The R script build_spliceu.R will build a spliceu (spliced + unspliced), an augmented transcriptome reference consisting of a augmented gene annotation GTF file, a transcriptome FASTA file, and a transcript-to-gene mapping TSV file.: "The R script `build_spliceu.R` will build a _spliceu_ (spliced + unspliced), an augmented transcriptome reference consisting of a augmented gene annotation GTF file, a transcriptome FASTA file, and a"
- [abstract] we show that our model can precisely predict the splicing status of many reads and identify the true gene origin of multi-gene mapped reads.: "we show that our model can precisely predict the splicing status of many reads and identify the true gene origin of multi-gene mapped reads."
- [readme] Then, we use samtools to filter the reads have unique map to the genome.: "Then, we use samtools to filter the reads have unique map to the genome."
