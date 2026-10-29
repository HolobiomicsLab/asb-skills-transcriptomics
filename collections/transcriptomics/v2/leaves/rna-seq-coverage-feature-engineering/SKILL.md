---
name: rna-seq-coverage-feature-engineering
description: Use when training or reproducing aptardi Mode 2 models that need RNA-Seq
  coverage (alone or with DNA sequence) as features over 3' terminal exons for polyadenylation-site
  prediction.
when_to_use_negative:
- The goal is de novo polyA detection with an existing pre-built model rather than
  engineering training features — use aptardi Mode 1 with the provided model.hdf5/scale.pk
  instead
- Analysis targets upstream (coding) exons for coding APA — aptardi only evaluates
  3' terminal exons
- Model must be reused with altered bin size — the pre-built model was built with
  100-base bins and altering --w is not recommended
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3518
tools:
- name: aptardi
  role: Builds the binned RNA-Seq coverage (and DNA sequence) feature matrix and trains
    the biLSTM polyA-site classifier in Mode 2
  repo: github.com/luskry/aptardi
- name: StringTie
  role: Reconstructs the input transcriptome (stringtie.gtf) whose 3' terminal exons
    define the coverage feature regions
- name: HISAT2
  role: Aligns RNA-Seq reads to hg38 to produce the BAM from which coverage features
    are binned
- name: SAMtools
  role: Sorts the aligned BAM required as aptardi's --b input (v.1.9 per README)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:8ae9b54dfb64ad482ff4dbb267693e8b73d24abdc758e00232ff79d5ed9c6fde
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HISAT2
  - SAMtools
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# rna-seq-coverage-feature-engineering

## Summary

Engineers sample-specific RNA-Seq read-coverage features for aptardi's machine-learning polyA-site model by binning sorted BAM coverage into 100-base bins over the 3' terminal exons of an input transcriptome (max 300 bins per transcript), optionally combined with DNA sequence features from hg38.fa.

## When to use

Use when training or reproducing aptardi Mode 2 models that need RNA-Seq coverage (alone or with DNA sequence) as features over 3' terminal exons for polyadenylation-site prediction.

## When NOT to use

- The goal is de novo polyA detection with an existing pre-built model rather than engineering training features — use aptardi Mode 1 with the provided model.hdf5/scale.pk instead
- Analysis targets upstream (coding) exons for coding APA — aptardi only evaluates 3' terminal exons
- Model must be reused with altered bin size — the pre-built model was built with 100-base bins and altering --w is not recommended

## Inputs

- Sorted BAM of genome-aligned RNA-Seq reads (e.g. HISAT2-aligned, SAMtools-sorted, Human Brain Reference RNA)
- Genome FASTA file (hg38.fa, headers = chromosomes)
- Input transcriptome GTF/GFF (e.g. stringtie.gtf) whose 3' terminal exons define the feature region
- Gold-standard polyA sites file (polya_sites.bed, tab-separated, columns given as 0-based indices via --l)

## Outputs

- Binned RNA-Seq coverage (and DNA sequence) feature matrix over 3' terminal exons in 100-base bins (max 300 bins/transcript)
- Trained Keras HDF5 model file (name_model.hdf5) and pickle scale file (name_scale.pk) per feature-set condition
- Per-condition test-set metrics: average precision, precision, recall, F-measure

## How to apply

Align RNA-Seq reads with HISAT2, sort the BAM with SAMtools, and reconstruct the transcriptome with StringTie (stringtie sorted.bam --rf -o stringtie.gtf). aptardi then bins RNA-Seq read coverage from the sorted BAM, together with DNA sequence from the FASTA (e.g. hg38.fa), into 100-base bins (--w default) over the 3' terminal exons of the input gtf, analyzing at most 300 bins per transcript (--i default, i.e. 30,000 bases). For multi-omics training run aptardi Mode 2 with --m, --s polya_sites.bed and --l 0,4,1 for gold-standard labels, saving a model (--e name_model.hdf5) and scale (--k name_scale.pk) per feature-set condition. To isolate RNA-only contributions, train with coverage features ablated of DNA-sequence features (or vice versa) under the same seed (--c) and compare test-set average precision; the article reports multi-omics AP = 0.58, RNA-only AP = 0.44, DNA-only AP = 0.41. Keep bin size at 100 when using any pre-built model, since it was built with 100-base bins.

## Related tools

- **aptardi** (Builds the binned RNA-Seq coverage (and DNA sequence) feature matrix and trains the biLSTM polyA-site classifier in Mode 2) — github.com/luskry/aptardi
- **StringTie** (Reconstructs the input transcriptome (stringtie.gtf) whose 3' terminal exons define the coverage feature regions)
- **HISAT2** (Aligns RNA-Seq reads to hg38 to produce the BAM from which coverage features are binned)
- **SAMtools** (Sorts the aligned BAM required as aptardi's --b input (v.1.9 per README))

## Examples

```
aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --m --s polya_sites.bed --l 0,4,1 --c 42 --e name_model.hdf5 --k name_scale.pk --o output_dir
```

## Evaluation signals

- Multi-omics model reaches AP ≈ 0.58 and F ≈ 0.51 on the held-out test split; DNA-only AP ≈ 0.41 and RNA-only AP ≈ 0.44
- Ablation conditions behave as expected: no-canonical-PAS AP ≈ 0.52 and canonical-PAS-only AP ≈ 0.54 relative to the full feature set
- Saved artifacts load correctly: model as Keras HDF5 and scale as pickle
- Repeat training with the same --c seed reproduces the same model, confirming reproducible feature construction and training
- Training- vs test-set AP estimates are similar, indicating aptardi is not overfitting on the engineered features

## Limitations

- Predictions are made on 100-base bins, a resolution limitation, although polyA sites can wiggle up to 30 nucleotides and few bins contain multiple sites
- Only 3' terminal exons of input transcripts are analyzed; upstream exons are not subjected to analysis, eliminating the possibility of identifying coding APA
- Manually engineered DNA sequence features may not apply outside mammals and require further research

## Evidence

- [readme] Bin size and transcript window defaults: "--i <int> Maximum length analyzed per transcript (default: 300, which is number of bins analyzed, i.e 300 and bin size of 100 = 300 * 100 = 30,000 bases"
- [readme] Mode 2 model-building requirements: "--m/-m <machine learning mode>	Enables Mode 2, building your own model, requires reliable genomic locations of polyA sites as the gold standard labels"
- [readme] Method scope over 3' terminal exons: "aptardi does not evaluate intron junctions but rather only evalutes 3' terminal exons of input transcripts and annotates 3' ends accordingly"
- [results] Multi-omics feature combination finding: [evidence span withheld — anchor status: not_found]
- [discussion] 100-base bin resolution caveat: "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
- [methods] Pipeline upstream of feature construction: "stringtie sorted.bam --rf -o stringtie.gtf -G <guide_file>"
