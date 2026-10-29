---
name: gold-standard-polya-site-labeling
description: Use when training a custom aptardi model (Mode 2) or benchmarking polyA
  site predictors against 3'-sequencing-derived gold-standard polyA sites that need
  to be mapped to label columns.
when_to_use_negative:
- You want to run aptardi's canonical Mode 1 with the pre-built HBR model (model.hdf5
  and scale.pk), which does not require gold-standard labels
- The polyA sites are unreliable or unvalidated — Mode 2 explicitly requires 'reliable
  genomic locations of polyA sites as the gold standard labels to train model'
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: aptardi
  role: Trains the biLSTM classifier in Mode 2 using the gold-standard polyA site
    labels supplied via --s/--l
  repo: github.com/luskry/aptardi
- name: StringTie
  role: Reconstructs the input transcriptome (stringtie.gtf) whose 3' terminal exons
    are evaluated against the gold-standard labels
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:3546b282d11cadcbd49b55c4292aaa9f8dba20c7476d9a340e5dc2c0c45d2298
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# gold-standard-polya-site-labeling

## Summary

Supply reliable gold-standard polyadenylation (polyA) site labels to train a custom aptardi Mode 2 model, using a tab-separated polyA sites file with 0-based chromosome, strand, and site column indices.

## When to use

Use when training a custom aptardi model (Mode 2) or benchmarking polyA site predictors against 3'-sequencing-derived gold-standard polyA sites that need to be mapped to label columns.

## When NOT to use

- You want to run aptardi's canonical Mode 1 with the pre-built HBR model (model.hdf5 and scale.pk), which does not require gold-standard labels
- The polyA sites are unreliable or unvalidated — Mode 2 explicitly requires 'reliable genomic locations of polyA sites as the gold standard labels to train model'

## Inputs

- Tab-separated gold-standard polyA sites file (e.g. polya_sites.bed) with chromosome, site, and strand columns
- 0-based comma-separated column indices for chromosome, strand, and site (e.g. 0,4,1)
- hg38.fa genome FASTA and sorted RNA-Seq BAM for feature extraction
- Input transcriptome GTF (e.g. stringtie.gtf)

## Outputs

- Custom-trained aptardi model file (name_model.hdf5, Keras HDF5)
- Custom scale file (name_scale.pk, Python pickle)
- Trained-model polyA site predictions and test-split metrics (average precision, precision, recall, F-measure)

## How to apply

Prepare a tab-separated file of reliable genomic polyA sites (e.g. from 3' sequencing such as PolyA-Seq-derived polya_sites.bed) to serve as gold-standard training labels. Invoke aptardi in Mode 2 by passing --m along with --s pointing to the labels file and --l giving the 0-based column indices for chromosome, site, and strand (e.g. --l 0,4,1 for polya_sites.bed). Set the seed with --c for reproducible model building and save the resulting Keras HDF5 model with --e and its pickle scale file with --k in the output directory. Train separate models per ablated feature-set condition (multi-omics, DNA-only, RNA-only, canonical-PAS ablations) to enable performance comparison via average precision, precision, recall, and F-measure on a held-out test split.

## Related tools

- **aptardi** (Trains the biLSTM classifier in Mode 2 using the gold-standard polyA site labels supplied via --s/--l) — github.com/luskry/aptardi
- **StringTie** (Reconstructs the input transcriptome (stringtie.gtf) whose 3' terminal exons are evaluated against the gold-standard labels)

## Examples

```
aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --m --s polya_sites.bed --l 0,4,1 --c 42 --e name_model.hdf5 --k name_scale.pk --o output_dir
```

## Evaluation signals

- The saved model file loads as a Keras HDF5 artifact and the scale file loads as a Python pickle
- Test-split metrics against the gold-standard sites reproduce reported values: multi-omics AP = 0.58 (F = 0.51), DNA-only AP = 0.41, RNA-only AP = 0.44, no-canonical-PAS AP = 0.52, canonical-PAS-only AP = 0.54
- Changing the seed via --c changes results while re-running with the same seed reproduces the model, confirming reproducible label-driven training
- The --l indices correctly resolve chromosome, strand, and site columns of the labels file (e.g. columns 0, 4, 1 of polya_sites.bed)

## Limitations

- Gold-standard polyA sites derived from one strain or species may not transfer: aptardi performed modestly worse on BNLx and SHR data sets, possibly because true sites came from Sprague Dawley rat
- aptardi predicts on 100 base bins, so label-site resolution is limited to the 3'-most base of a predicted bin (polyA sites can wiggle up to 30 nucleotides)
- aptardi does not distinguish transcripts overlapping the same region, so one labeled site may be attributed to multiple transcripts of a gene

## Evidence

- [readme] Mode 2 requires reliable labels: "--m/-m <machine learning mode>	Enables Mode 2, building your own model, requires reliable genomic locations of polyA sites as the gold standard labels to train model"
- [readme] Label column indices: "--l/-l <int,int,int>		0-based coordinates of chromosome, strand, and site columns in polyA sites file (comma separated list with no spaces)"
- [readme] Seed for reproducibility: "--c/-c <int>			Set seed for reproducibly building model"
- [intro] Training objective from 3' sequencing: "we trained aptardi to detect polyA sites from DNA nucleotide sequence and RNA-Seq read coverage by training on polyA sites identified by 3′ sequencing"
- [discussion] Bin resolution limitation: "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
