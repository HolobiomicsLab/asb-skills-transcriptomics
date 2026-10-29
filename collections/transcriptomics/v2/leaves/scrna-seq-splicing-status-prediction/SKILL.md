---
name: scrna-seq-splicing-status-prediction
description: Use when scRNA-seq reads labeled 'ambiguous' (compatible with both spliced
  and unspliced transcripts) need a probabilistic spliced/unspliced resolution, e.g.
  PBMC1k-style reads aligned to a spliceu augmented reference.
when_to_use_negative:
- Reads already carry reliable spliced/unspliced labels (i.e. not from single-cell
  data with ambiguous counts) — Forseti is designed for scRNA-seq reads compatible
  with both spliced and unspliced transcripts
- Single-end reads or alignments without transcriptomic coordinates — Forseti scores
  putative fragments that associate each alignment with proximate priming sites, requiring
  toTranscriptome BAM output from STAR
- Long-read or non-10X data where oligo(dT) priming assumptions and the 10X whitelist
  (e.g. 3M-february-2018.txt) do not apply
edam_operation: http://edamontology.org/operation_0433
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Forseti
  role: Python package providing forseti_predictor, the master function that assigns
    probabilistic splicing status to reads using the binding-affinity and fragment-length
    models
  repo: https://github.com/COMBINE-lab/forseti
- name: STAR
  role: Aligns reads to the spliceu index and emits toTranscriptome BAM alignments
    in transcript coordinates
- name: samtools
  role: Filters reads with unique genome mapping via the -d NH:1 flag
- name: bedtools
  role: bedtools intersect (-f 1.0) identifies reads 100% mapped in exonic regions
    (ambiguous reads)
- name: R
  role: Runs build_spliceu.R to construct the spliceu augmented reference from genome
    FASTA and GTF
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:9c8bd865a9421b82896d8d940745484a7690aa3ece4a3eee139766e3ce7c2cbf
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
  edam_operation: http://edamontology.org/operation_0433
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# scrna-seq-splicing-status-prediction

## Summary

Apply Forseti, a mechanistic predictive model, to probabilistically assign a splicing status (spliced, unspliced, or ambiguous) to scRNA-seq reads by scoring putative fragments against proximate oligo(dT) priming sites using a trained binding-affinity MLP and a cubic-spline fragment-length distribution model.

## When to use

Use when scRNA-seq reads labeled 'ambiguous' (compatible with both spliced and unspliced transcripts) need a probabilistic spliced/unspliced resolution, e.g. PBMC1k-style reads aligned to a spliceu augmented reference.

## When NOT to use

- Reads already carry reliable spliced/unspliced labels (i.e. not from single-cell data with ambiguous counts) — Forseti is designed for scRNA-seq reads compatible with both spliced and unspliced transcripts
- Single-end reads or alignments without transcriptomic coordinates — Forseti scores putative fragments that associate each alignment with proximate priming sites, requiring toTranscriptome BAM output from STAR
- Long-read or non-10X data where oligo(dT) priming assumptions and the 10X whitelist (e.g. 3M-february-2018.txt) do not apply

## Inputs

- toTranscriptome BAM from STAR (--quantMode TranscriptomeSAM) aligned to a spliceu reference
- Uniquely genome-mapped read-name list (samtools -d NH:1 filter)
- spliceu.fa augmented transcriptome FASTA
- t2g_3col.tsv transcript-to-gene mapping
- Pre-trained models: spline_model.pkl (fragment-length cubic spline) and mlp_pa6with1mm.pkl (oligo(dT) binding-affinity MLP)
- 10X scRNA-seq FASTQs (e.g. PBMC1k v3) plus genome FASTA and GTF for spliceu construction

## Outputs

- forseti_dict: per-read splicing status ('S' spliced, 'U' unspliced, 'A' ambiguous) with splicing probabilities
- ROC curves and AUC scores on evaluation read sets (reported: mean 0.92 simulated, 0.88 experimental)

## How to apply

First build a spliceu (spliced + unspliced) augmented transcriptome reference — spliceu.gtf, spliceu.fa, t2g_3col.tsv — from a genome FASTA and gene GTF (e.g. refdata-gex-GRCh38-2020-A) with build_spliceu.R. Align reads with STAR (--runThreadN 12, --quantMode TranscriptomeSAM, --outSAMtype BAM SortedByCoordinate), then filter uniquely genome-mapped reads using samtools with the -d NH:1 flag and extract toTranscriptome alignments, marking fully exonic reads with bedtools intersect (-f 1.0). Import the forseti Python package and call the master function forseti_predictor from forseti.modules.prediction_model with the toTranscriptome BAM, uniquely-mapped read-name list, spliceu.fa, t2g_3col.tsv, and the pre-trained models spline_model.pkl and mlp_pa6with1mm.pkl, using snr_min_size=6, discount_perc=1, polya_tail_len=200, max_frag_len=1000, and num_threads=12. Parse the returned forseti_dict, which contains each read's final splicing_status ('S', 'U', or 'A') and prediction probabilities; ambiguous reads whose status stays 'A' are those Forseti cannot mechanistically resolve (fundamental ambiguity). Validate on data with ground truth by excluding reads retained as 'A' and computing ROC AUC — expect ~0.92 mean on simulated sets and ~0.88 on experimental data.

## Related tools

- **Forseti** (Python package providing forseti_predictor, the master function that assigns probabilistic splicing status to reads using the binding-affinity and fragment-length models) — https://github.com/COMBINE-lab/forseti
- **STAR** (Aligns reads to the spliceu index and emits toTranscriptome BAM alignments in transcript coordinates)
- **samtools** (Filters reads with unique genome mapping via the -d NH:1 flag)
- **bedtools** (bedtools intersect (-f 1.0) identifies reads 100% mapped in exonic regions (ambiguous reads))
- **R** (Runs build_spliceu.R to construct the spliceu augmented reference from genome FASTA and GTF)

## Examples

```
from forseti.modules.prediction_model import forseti_predictor; forseti_dict = forseti_predictor(toTranscriptome.bam, unique_read_names, spliceu.fa, t2g_3col.tsv, 'spline_model.pkl', 'mlp_pa6with1mm.pkl', snr_min_size=6, discount_perc=1, polya_tail_len=200, max_frag_len=1000, num_threads=12)
```

## Evaluation signals

- On simulated read sets (with and without fundamental ambiguity) the computed AUC on reads not retained as 'A' falls in the range 0.85–0.93 with mean ≈0.92
- On experimental evaluation datasets AUC ≈ 0.88; trivial baselines ('All spliced', 'All unspliced', 'All random') attain exactly AUC = 0.5, so any score near 0.5 signals a broken pipeline
- forseti_dict contains a final splicing_status for every input read, with status values restricted to 'S', 'U', or 'A'
- Fully exonic reads identified by bedtools intersect (-f 1.0) match the reads Forseti flags as ambiguity candidates
- STAR run completed with --quantMode TranscriptomeSAM producing toTranscriptome BAM, and samtools NH:1 filtering yielded the uniquely-mapped read-name list passed to forseti_predictor

## Limitations

- As a mechanistic model, Forseti cannot distinguish fundamentally ambiguous reads (polyA tail priming with read2 within the terminal exon, vs. cDNA fragments arising entirely from an exon), so some reads remain 'A' after prediction
- Prediction is computationally heavy: the filtered PBMC1k top-200-cell BAM (65,784,347 alignments) took ~3.3 hours on 32 threads
- Performance depends on pre-trained models (spline_model.pkl, mlp_pa6with1mm.pkl) whose binding-affinity MLP was trained on empirical priming windows; transfer to tissues/species relies on the fragment-length model generalizing across datasets
- The tutorial pipeline assumes 10X chemistry v3 conventions (--soloUMIlen 12, 3M-february-2018.txt whitelist); other chemistries need adjusted STAR --solo* parameters

## Evidence

- [abstract] AUC on simulated vs experimental data: "Forseti obtains a mean Area Under the Receiver Operating Characteristic Curve (AUC) score of 0.92 on simulated data, and 0.88 on the experimental data."
- [abstract] model components: [evidence span withheld — anchor status: not_found]
- [methods] ambiguous read definition: "Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts. Our prediction model processes these ambiguous reads and accurately predict splicing status"
- [methods] predictor entry point: "call the master function named `forseti_predictor` for the splicing status prediction"
- [methods] unique mapping filter: "Then, we use samtools to filter the reads have unique map to the genome."
- [readme] python import: "Forseti is implemented in Python and you could easily import the `forseti` package"
- [readme] fundamental ambiguity scope: [evidence span withheld — anchor status: external_doc]
