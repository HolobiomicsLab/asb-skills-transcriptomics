---
name: genomic-sequence-bin-feature-extraction
description: Use when you need per-position (binned) DNA sequence and/or RNA-Seq coverage
  features over transcript 3' terminal exons from a GTF to feed a sequence+coverage
  machine-learning model such as aptardi's biLSTM.
when_to_use_negative:
- Your goal is full transcript assembly including intron junctions — aptardi only
  evaluates 3' terminal exons of the input transcripts and does not evaluate intron
  junctions
- You need resolution finer than the bin width for polyA site localization — aptardi
  annotates only the 3'-most base of a positive 100-base bin, while true sites wiggle
  up to 30 nucleotides
- Your input is already a binned feature table or you are using the pre-built HBR
  model with a non-default bin size — the pre-built model requires 100-base bins
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0654
tools:
- name: aptardi
  role: Performs the 100-base binning of DNA sequence and RNA-Seq coverage over 3'
    terminal exons and trains/evaluates the biLSTM model on the binned features
  repo: github.com/luskry/aptardi
- name: StringTie
  role: Reconstructs the input transcriptome (stringtie.gtf) whose 3' terminal exons
    define the regions to be binned
- name: SAMtools
  role: Produces the sorted BAM of HISAT2-aligned RNA-Seq reads from which per-bin
    coverage is extracted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:3216f69bc8bf518c13bb3567b793a5ba9ce23fe9add242e1a2a92471035f8961
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SAMtools
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# genomic-sequence-bin-feature-extraction

## Summary

Builds fixed-width binned feature representations of DNA sequence (and aligned RNA-Seq coverage) over transcript regions, as implemented in aptardi, which bins the genome into 100-base windows spanning the 3' terminal exons of an input transcriptome to train a biLSTM polyadenylation-site classifier.

## When to use

Use when you need per-position (binned) DNA sequence and/or RNA-Seq coverage features over transcript 3' terminal exons from a GTF to feed a sequence+coverage machine-learning model such as aptardi's biLSTM.

## When NOT to use

- Your goal is full transcript assembly including intron junctions — aptardi only evaluates 3' terminal exons of the input transcripts and does not evaluate intron junctions
- You need resolution finer than the bin width for polyA site localization — aptardi annotates only the 3'-most base of a positive 100-base bin, while true sites wiggle up to 30 nucleotides
- Your input is already a binned feature table or you are using the pre-built HBR model with a non-default bin size — the pre-built model requires 100-base bins

## Inputs

- Reference genome FASTA (e.g. hg38.fa; headers must be chromosomes)
- Sorted, genome-aligned RNA-Seq BAM file
- Input transcriptome in GTF/GFF format (designed for StringTie output) defining 3' terminal exons
- Tab-separated gold-standard polyA sites BED file with 0-based column indices for chromosome, strand, and site (e.g. --l 0,4,1) for supervised training

## Outputs

- Binned feature matrix (DNA sequence + RNA-Seq coverage features per 100-base bin, up to 300 bins per transcript)
- Trained Keras model file (.hdf5) per feature-set condition
- Feature scaler pickle file (.pk) per feature-set condition
- Modified sample-specific transcriptome GTF with predicted polyA sites annotating 3' ends

## How to apply

Binned feature extraction in aptardi works by taking the 3' terminal exons of transcripts in an input GTF (designed for StringTie output), partitioning each into fixed 100-base bins (the --w default; choices are 25-200 in 25 base increments), and encoding DNA sequence from a hg38 FASTA plus read coverage from a sorted BAM into up to 300 bins per transcript (--i default, i.e. 30,000 bases analyzed per transcript). Rationale: 100-base bins balance resolution with the fact that polyA sites wiggle up to ~30 nucleotides, and predictions annotate the 3'-most base of a positive bin. Do not alter --w when using the pre-built HBR model, since it was trained on 100-base bins. The resulting binned feature matrix is the input to Mode 2 model training; the trained model (Keras HDF5) and its feature scaler (pickle) are saved per experimental condition.

## Related tools

- **aptardi** (Performs the 100-base binning of DNA sequence and RNA-Seq coverage over 3' terminal exons and trains/evaluates the biLSTM model on the binned features) — github.com/luskry/aptardi
- **StringTie** (Reconstructs the input transcriptome (stringtie.gtf) whose 3' terminal exons define the regions to be binned)
- **SAMtools** (Produces the sorted BAM of HISAT2-aligned RNA-Seq reads from which per-bin coverage is extracted)

## Examples

```
aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --m --s polya_sites.bed --l 0,4,1 --c 42 --e name_model.hdf5 --k name_scale.pk --o output_dir
```

## Evaluation signals

- The binned region spans only 3' terminal exons of the input GTF, with bin size fixed at 100 bases (or explicitly overridden via --w in 25-base increments) and a maximum of 300 bins (--i, i.e. 30,000 bases) per transcript
- Saved artifacts validate as a Keras-loadable HDF5 model and a pickle-loadable scale file when training in Mode 2
- Held-out test performance of the multi-omics binned model should reach AP ≈ 0.58 (vs DNA-only ≈ 0.41, RNA-only ≈ 0.44), demonstrating the binned sequence+coverage features carry complementary signal
- Positive predictions annotate the transcript stop site as the 3'-most base of the predicted bin, and few bins should contain multiple gold-standard sites given the ≤30 nt wiggle of polyA sites

## Limitations

- 100-base bin resolution limits localization accuracy: aptardi annotates the 3'-most base of a predicted bin although polyA sites can wiggle up to 30 nucleotides
- Only 3' terminal exons are analyzed, so coding alternative polyadenylation in upstream exons cannot be identified
- Bin size (--w) must remain at the default 100 when using the pre-built HBR model, because that model was trained on 100-base bins
- Manually engineered DNA sequence features may not transfer to taxa outside of mammals

## Evidence

- [readme] bin width parameterization: "--w/-w <int>			Size of bin to make predictions on (default: 100, choices: 25-200 in 25 base increments"
- [readme] 3' terminal exon scope: "aptardi does not evaluate intron junctions but rather only evalutes 3' terminal exons of input transcripts and annotates 3' ends accordingly."
- [discussion] bin-level annotation semantics: "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
- [abstract] input triad: "as input aptardi takes DNA nucleotide sequence, genome-aligned RNA-Seq data, and an initial transcriptome"
