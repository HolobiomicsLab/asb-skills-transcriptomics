---
name: single-cell-read-splicing-status-prediction
description: Use when single-cell RNA-seq reads aligned to a transcriptome need their
  spliced/unspliced status predicted, or when multi-gene mapped reads must be resolved
  to a single gene of origin.
when_to_use_negative:
- Reads are not single-cell scRNA-seq alignments against a spliceu augmented reference
  (the model scores oligo(dT) priming sites and fragment lengths specific to scRNA-seq).
- Reads that map uniquely to a single gene with no multi-gene ambiguity — splicing
  status prediction is the model's focus, and multi-gene rescue requires tied best
  genes.
- You need to distinguish fundamentally ambiguous reads arising from polyA tail priming
  within the terminal exon versus exon-derived cDNA fragments — Forseti as a mechanistic
  model cannot separate these two sources.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Forseti
  role: Master forseti_predictor function scores putative fragments to predict splicing
    status of scRNA-seq reads
  repo: https://github.com/COMBINE-lab/forseti
- name: STAR
  role: Aligns reads against the spliceu genome index with TranscriptomeSAM quantification
- name: samtools
  role: Filters uniquely genome-mapped reads via the NH:1 flag
- name: bedtools
  role: Intersect to find reads 100% mapped in exonic regions (ambiguous reads)
- name: conda
  role: Creates build_spliceu_r and forseti_test environments from yml files
- name: R
  role: Runs build_spliceu.R to construct the spliceu augmented reference
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:1222bf989f797af34108163c58b80c06220b342835f7fa60cedd7060b430fbd7
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
  - conda
  - samtools
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-read-splicing-status-prediction

## Summary

Apply Forseti, a mechanistic model combining a trained oligo(dT) binding-affinity MLP and a cubic-spline fragment length distribution, to probabilistically assign splicing status and gene origin to scRNA-seq reads.

## When to use

Use when single-cell RNA-seq reads aligned to a transcriptome need their spliced/unspliced status predicted, or when multi-gene mapped reads must be resolved to a single best gene of origin.

## When NOT to use

- Reads are not single-cell scRNA-seq alignments against a spliceu augmented reference (the model scores oligo(dT) priming sites and fragment lengths specific to scRNA-seq).
- Reads that map uniquely to a single gene with no multi-gene ambiguity — splicing status prediction is the model's focus, and multi-gene rescue requires tied best genes.
- You need to distinguish fundamentally ambiguous reads arising from polyA tail priming within the terminal exon versus exon-derived cDNA fragments — Forseti as a mechanistic model cannot separate these two sources.

## Inputs

- spliced + unspliced (spliceu) reference: spliceu.fa, spliceu.gtf, t2g_3col.tsv
- transcriptome-aligned BAM from STAR (Aligned.toTranscriptome.out.bam)
- unimap_read_names.txt list of uniquely genome-mapped reads (samtools NH:1 filter)
- pre-trained models: spline_model.pkl (fragment length distribution) and mlp_pa6with1mm.pkl (binding affinity MLP)
- paired-end scRNA-seq FASTQ files (e.g. 10x PBMC1k v3)

## Outputs

- forseti_dict with per-read splicing status predictions
- predictions_log with read_count, final_multi_gene_count, rescued_multigene_count, and final_ambiguous_count
- multi-gene assignment metrics (e.g. multigene_assignment_metrics.csv)

## How to apply

Build a spliceu (spliced + unspliced) augmented reference with build_spliceu.R from a genome FASTA and gene GTF, producing spliceu.fa and t2g_3col.tsv. Align reads with STAR using --quantMode TranscriptomeSAM and --soloFeatures GeneFull, then filter uniquely genome-mapped reads with samtools using the NH:1 flag. Activate the forseti_test conda environment and call the master function forseti_predictor with the default parameter set (snr_min_size=6, discount_perc=1, polya_tail_len=200, max_frag_len=1000, num_threads=12), supplying the pre-built spline_model.pkl and mlp_pa6with1mm.pkl models. For multi-gene mapped reads (is_multi_mapped False, tied best genes), apply the maximum-score gene-origin criterion to rescue reads as single-gene assignments; target rates are 65% and 93% assigned and 75% and 69% correct on the two experimental evaluation datasets.

## Related tools

- **Forseti** (Master forseti_predictor function scores putative fragments to predict splicing status of scRNA-seq reads) — https://github.com/COMBINE-lab/forseti
- **STAR** (Aligns reads against the spliceu genome index with TranscriptomeSAM quantification)
- **samtools** (Filters uniquely genome-mapped reads via the NH:1 flag)
- **bedtools** (Intersect to find reads 100% mapped in exonic regions (ambiguous reads))
- **conda** (Creates build_spliceu_r and forseti_test environments from yml files)
- **R** (Runs build_spliceu.R to construct the spliceu augmented reference)

## Examples

```
from forseti import forseti_predictor; forseti_dict, predictions_log = forseti_predictor(spliceu_fa='spliceu.fa', t2g='t2g_3col.tsv', bam='Aligned.toTranscriptome.out.bam', unimap_read_names='unimap_read_names.txt', snr_min_size=6, discount_perc=1, polya_tail_len=200, max_frag_len=1000, num_threads=12)
```

## Evaluation signals

- Mean AUC of 0.92 on simulated data and 0.88 on experimental data for splicing status prediction
- Multi-gene assignment rates of 65% and 93% (single best gene origin) on the two experimental evaluation datasets, matching Section 3.3 values
- Conditional correctness of 75% and 69% among rescued multi-gene assignments
- predictions_log fields (read_count, final_multi_gene_count, rescued_multigene_count, final_ambiguous_count) sum consistently over reads in forseti_dict
- Filtered BAM for the 10X 1kPBMC top-200-cell tutorial contains 65,784,347 alignments

## Limitations

- As a mechanistic model, Forseti cannot distinguish fundamentally ambiguous reads: those from polyA tail priming with read2 inside the terminal exon versus reads whose cDNA fragment arose from an exon.
- Multi-gene mapped reads whose highest score is tied across genes remain ambiguous (final_ambiguous_count).
- The full 10X 1kPBMC dataset has ~200 million transcriptome alignments; prediction took 3.3 hours on 32 threads even after filtering to the top 200 cells.

## Evidence

- [abstract] Forseti obtains a mean Area Under the Receiver Operating Characteristic Curve (AUC) score of 0.92 on simulated data, and 0.88 on the experimental data.: "Forseti obtains a mean Area Under the Receiver Operating Characteristic Curve (AUC) score of 0.92 on simulated data, and 0.88 on the experimental data."
- [methods] forseti_predictor call: "call the master function named `forseti_predictor` for the splicing status prediction"
- [methods] NH:1 filter: "# filter genome unique mapped reads 
# `-d NH:1` with this flag, we get the uniquely mapped reads"
- [methods] ambiguous reads: "Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts. Our prediction model processes these ambiguous reads and accurately predict splicing status"
- [readme] Forseti model description: "**_Forseti_** is a predictive model to probabilistically assign a splicing status to scRNA-seq reads."
