---
name: polya-priming-window-scoring
description: Use when you need to predict the splicing status (spliced, unspliced,
  or ambiguous 'A') of individual scRNA-seq reads, especially reads aligned to a spliceu
  augmented reference and filtered to unique genome mappings.
when_to_use_negative:
- You only need gene- or transcript-level abundance quantification rather than per-read
  splicing status — a standard pseudoaligner/quantifier is the wrong tool.
- The reads are bulk RNA-seq rather than single-cell scRNA-seq; the binding affinity
  and fragment length models were trained on scRNA-seq priming behavior.
- No spliceu (spliced + unspliced augmented) reference or trained model pickles are
  available and you cannot build/train them.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Forseti
  role: Provides forseti_predictor, the master function scoring priming windows and
    fragment lengths to assign splicing status
  repo: https://github.com/COMBINE-lab/forseti
- name: STAR
  role: Aligns reads to the spliceu genome index with --quantMode TranscriptomeSAM
    producing the toTranscriptome BAM input
- name: samtools
  role: Filters reads that uniquely map to the genome via the -d NH:1 flag
- name: bedtools
  role: Marks fully exonic (100% exonic-region) reads with bedtools intersect -f 1.0
    to identify ambiguous reads
- name: Python
  role: Language in which the forseti package and prediction model are implemented
    and invoked
  repo: https://github.com/COMBINE-lab/forseti
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:fe837205263370edab9990ae1742f5ef7a93d34a8e6a3cd39c0ade97474739c2
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# polya-priming-window-scoring

## Summary

Score putative priming windows for oligo(dT) primers in scRNA-seq reads using Forseti's trained binding affinity MLP (mlp_pa6with1mm.pkl) together with a cubic-spline fragment length distribution model, to probabilistically assign a splicing status to reads aligned to a spliceu reference.

## When to use

Use when you need to predict the splicing status (spliced, unspliced, or ambiguous 'A') of individual scRNA-seq reads, especially reads aligned to a spliceu augmented reference and filtered to unique genome mappings.

## When NOT to use

- You only need gene- or transcript-level abundance quantification rather than per-read splicing status — a standard pseudoaligner/quantifier is the wrong tool.
- The reads are bulk RNA-seq rather than single-cell scRNA-seq; the binding affinity and fragment length models were trained on scRNA-seq priming behavior.
- No spliceu (spliced + unspliced augmented) reference or trained model pickles are available and you cannot build/train them.

## Inputs

- toTranscriptome BAM from STAR alignment to spliceu reference
- List of uniquely genome-mapped read names (samtools -d NH:1)
- spliceu.fa transcriptome FASTA
- t2g_3col.tsv transcript-to-gene mapping
- Pre-trained binding affinity model mlp_pa6with1mm.pkl
- Pre-trained fragment length distribution spline_model.pkl

## Outputs

- forseti_dict mapping reads to predicted splicing status (spliced, unspliced, or 'A' ambiguous)
- Per-read splicing-status predictions usable for ROC/AUC evaluation

## How to apply

Align reads to the spliceu augmented reference (spliced + unspliced transcriptome built by build_spliceu.R) with STAR using --quantMode TranscriptomeSAM and --outSAMtype BAM SortedByCoordinate. Filter uniquely genome-mapped reads with samtools (-d NH:1), extract toTranscriptome alignments, and mark fully exonic reads with bedtools intersect (-f 1.0). Call the Python master function forseti_predictor from forseti.modules.prediction_model with the toTranscriptome BAM, uniquely-mapped read-name list, spliceu.fa, t2g_3col.tsv, and the pre-trained models spline_model.pkl and mlp_pa6with1mm.pkl, using parameters snr_min_size=6, discount_perc=1, polya_tail_len=200, max_frag_len=1000, and num_threads=12. Forseti scores putative fragments associating each read alignment with proximate potential priming sites, combining the MLP binding-affinity score with the cubic-spline fragment length distribution; parse the returned forseti_dict and treat reads with final splicing_status 'A' as compatible with both spliced and unspliced transcripts.

## Related tools

- **Forseti** (Provides forseti_predictor, the master function scoring priming windows and fragment lengths to assign splicing status) — https://github.com/COMBINE-lab/forseti
- **STAR** (Aligns reads to the spliceu genome index with --quantMode TranscriptomeSAM producing the toTranscriptome BAM input)
- **samtools** (Filters reads that uniquely map to the genome via the -d NH:1 flag)
- **bedtools** (Marks fully exonic (100% exonic-region) reads with bedtools intersect -f 1.0 to identify ambiguous reads)
- **Python** (Language in which the forseti package and prediction model are implemented and invoked) — https://github.com/COMBINE-lab/forseti

## Examples

```
from forseti.modules.prediction_model import forseti_predictor; result = forseti_predictor('Aligned.toTranscriptome.out.bam', unique_reads, 'spliceu.fa', 't2g_3col.tsv', 'spline_model.pkl', 'mlp_pa6with1mm.pkl', snr_min_size=6, discount_perc=1, polya_tail_len=200, max_frag_len=1000, num_threads=12)
```

## Evaluation signals

- On the two simulated evaluation read sets (with and without fundamental ambiguity), excluding reads with final splicing_status 'A' should yield AUC values in the range 0.85-0.93 with mean ~0.90-0.92.
- On experimental data, mean AUC should be near the reported 0.88.
- The forseti_dict output must contain a final splicing_status key for each read, with values limited to the model's status labels including 'A' for ambiguous.
- Reads marked as 100% exonic by bedtools intersect (-f 1.0) should correspond to reads the model flags or processes as ambiguous ('A') candidates.
- The uniquely-mapped read-name list from samtools -d NH:1 should match one-to-one with the reads present in the toTranscriptome BAM used as forseti_predictor input.

## Limitations

- Forseti is a mechanistic model and cannot distinguish fundamentally ambiguous reads — those arising from polyA tail priming with read2 contained in the terminal exon, or reads whose cDNA fragment arises entirely from an exon.
- Model accuracy depends on the pre-trained MLP (mlp_pa6with1mm.pkl) and spline (spline_model.pkl); the fragment length distribution model must generalize across species and tissue types.
- Prediction on large datasets is slow: the filtered PBMC1k BAM containing 65,784,347 alignments took 3.3 hours on 32 threads, so extracting top cells may be needed for testing.

## Evidence

- [intro] first, we train a binding affinity model to assign a probability that a given transcriptomic site is used in fragment generation: "first, we train a binding affinity model to assign a probability that a given transcriptomic site is used in fragment generation"
- [readme] Forseti combines these two trained models to predict the splicing status of the molecule of origin of reads by scoring putative fragments that associate each alignment of sequenced reads with proximate potential priming sites.: "**_Forseti_** combines these two trained models to predict the splicing status of the molecule of origin of reads by scoring putative fragments that associate each alignment of sequenced reads with"
- [methods] Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts. Our prediction model processes these ambiguous reads and accurately predict splicing status: "Ambiguous reads ('A') are those compatible with both spliced and unspliced transcripts. Our prediction model processes these ambiguous reads and accurately predict splicing status"
- [readme] we train a multi-layer perceptron (MLP) using the priming windows to predict the binding affinity of oligo(dT) primer for given sequences: [evidence span withheld — anchor status: external_doc]
- [readme] fundamentally ambiguious reads are the reads that come from either polyA tail priming and the read2 is contained within the terminal exon, or the reads where the original cDNA fragment arising from an exon.: [evidence span withheld — anchor status: external_doc]
