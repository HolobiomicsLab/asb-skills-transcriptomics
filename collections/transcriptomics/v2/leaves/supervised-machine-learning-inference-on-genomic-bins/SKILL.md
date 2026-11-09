---
name: supervised-machine-learning-inference-on-genomic-bins
description: Use when you have a sample-specific transcriptome (GTF), genome-aligned
  RNA-Seq (sorted BAM), and genome FASTA, and need de novo polyA site prediction via
  machine learning rather than assembler coverage heuristics or annotated polyA site
  databases.
when_to_use_negative:
- Your goal is detecting coding alternative polyadenylation in upstream exons — aptardi
  only analyzes 3' terminal exons of input transcripts
- You need to change the bin size from 100 bases when using the pre-built model, which
  was trained exclusively on 100-base bins
- Your organism/taxon is outside mammals — the manually engineered DNA sequence features
  may not transfer
edam_operation: http://edamontology.org/operation_0428
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: aptardi
  role: Runs the supervised ML inference over binned 3'-terminal exon features to
    predict polyA sites and emit the modified transcriptome GTF
  repo: https://github.com/luskry/aptardi
- name: SAMtools
  role: Sorts the HISAT2-aligned BAM so bins can extract RNA-Seq coverage features;
    must be installed outside the aptardi conda environment
- name: BEDtools
  role: Supports genomic interval operations during feature extraction; must be installed
    outside the aptardi conda environment
- name: StringTie
  role: Produces the input stringtie.gtf transcriptome whose 3' terminal exons aptardi
    featurizes and scores
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:6dcf4c87e6b2edeab3220061fedb545f099bb85b359e42b83091a860bbdef2dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - BEDtools
  - SAMtools
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_0428
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# supervised-machine-learning-inference-on-genomic-bins

## Summary

Apply aptardi's pre-trained supervised model (model.hdf5 + scale.pk) to run ML inference over 100-base genomic bins of 3'-terminal exons, classifying bins as transcription stop sites by combining RNA-Seq coverage features with DNA sequence features.

## When to use

Use when you have a sample-specific transcriptome (GTF), genome-aligned RNA-Seq (sorted BAM), and genome FASTA, and need de novo polyA site prediction via machine learning rather than assembler coverage heuristics or annotated polyA site databases.

## When NOT to use

- Your goal is detecting coding alternative polyadenylation in upstream exons — aptardi only analyzes 3' terminal exons of input transcripts
- You need to change the bin size from 100 bases when using the pre-built model, which was trained exclusively on 100-base bins
- Your organism/taxon is outside mammals — the manually engineered DNA sequence features may not transfer

## Inputs

- sorted BAM of genome-aligned RNA-Seq reads (e.g. HISAT2-aligned, SAMtools-sorted)
- unzipped genome FASTA with chromosome headers (e.g. hg38.fa)
- input transcriptome in GTF/GFF format (StringTie output recommended, e.g. stringtie.gtf)
- pre-trained model file model.hdf5 from src/aptardi/ml_scale
- pre-trained scaler file scale.pk from src/aptardi/ml_scale

## Outputs

- modified transcriptome GTF/GFF (e.g. name_aptardi.gtf) containing original input transcripts plus added aptardi transcripts with predicted 3' ends
- predicted polyA (transcription stop) sites called at probability threshold 0.5, annotated as the 3'-most base of positive 100-base bins

## How to apply

Run aptardi in Mode 1 (canonical usage) after installing via conda in an isolated environment — the bioconda samtools and bedtools packages must NOT share the environment with aptardi or it may fail. Provide --b a sorted BAM of HISAT2-aligned RNA-Seq reads, --f an unzipped hg38 FASTA, --r a GTF/GFF transcriptome (designed for StringTie output), and --n/--t pointing to the pre-built HBR model.hdf5 and scale.pk from src/aptardi/ml_scale. The pipeline bins each input transcript's 3'-terminal exon into 100-base increments (bin size --w 100 must not be altered with the pre-built model) and computes 27 features per bin: 14 RNA-Seq features, 12 DNA sequence features, and one from the original transcriptome; it analyzes up to 300 bins (30,000 bases) per transcript and only evaluates 3' terminal exons, never intron junctions. Bins with predicted probability >= 0.5 (--p 0.5) are labeled transcription stop sites, annotated as the 3'-most base of the positive bin; set --a fr for first-strand Illumina paired-end libraries. Output is a new GTF/GFF (e.g. name_aptardi.gtf) containing the unmodified input transcripts plus aptardi transcripts — aptardi removes nothing from the input transcriptome.

## Related tools

- **aptardi** (Runs the supervised ML inference over binned 3'-terminal exon features to predict polyA sites and emit the modified transcriptome GTF) — https://github.com/luskry/aptardi
- **SAMtools** (Sorts the HISAT2-aligned BAM so bins can extract RNA-Seq coverage features; must be installed outside the aptardi conda environment)
- **BEDtools** (Supports genomic interval operations during feature extraction; must be installed outside the aptardi conda environment)
- **StringTie** (Produces the input stringtie.gtf transcriptome whose 3' terminal exons aptardi featurizes and scores)

## Examples

```
aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --n model.hdf5 --t scale.pk --g name_aptardi.gtf --o output_dir --w 100 --i 300 --p 0.5 --a fr
```

## Evaluation signals

- aptardi --version, samtools --version, and bedtools --version all succeed from within the activated conda environment
- Output GTF (name_aptardi.gtf) parses as valid GTF/GFF and contains the unmodified input transcripts plus added aptardi transcripts (no transcripts removed)
- Predicted 3' ends coincide with the 3'-most base of 100-base bins whose model probability >= 0.5
- Pipeline evaluated only 3' terminal exons of input transcripts (no intron-junction assessment, no upstream exon predictions)
- Downstream RSEM quantitation of the aptardi transcriptome yields transcripts; filtering (>=1 count in 2/3 of samples) removes redundant/false-positive aptardi transcripts from isoform-dense loci

## Limitations

- Predictions have 100-base bin resolution; for positive predictions aptardi annotates the 3'-most base of the bin, though polyA sites can wiggle up to 30 nucleotides and few bins contained multiple sites
- aptardi incorporates a single 3'-end into multiple transcripts per gene because it does not distinguish transcripts overlapping the same genomic region, possibly attributing a stop site to the wrong transcript
- Upstream exons are not analyzed, eliminating the possibility of identifying coding APA
- Manually engineered DNA sequence features may not apply outside mammals and require further research
- Performance was modestly worse on BNLx and SHR datasets, possibly owing to unstranded RNA-Seq and/or different species

## Evidence

- [methods] bins the modified 3'-terminal exons of transcript models into 100-base increments and computes 27 features per bin: 14 RNA-Seq features, 12 DNA sequence-related features, and one feature derived from the original transcriptome: [evidence span withheld — anchor status: not_found]
- [readme] apardi takes GTF/GFF input and annotates 3' ends only on terminal exons: "Note that aptardi does not evaluate intron junctions but rather only evalutes 3' terminal exons of input transcripts and annotates 3' ends accordingly."
- [readme] bioconda samtools and bedtools conflict warning: "The bioconda samtools and bedtools packages may prevent aptardi from working, please make sure these packages are not installed in the same environment as aptardi"
- [readme] default thresholds and binning parameters: "--p/-p <float>			Probability threshold, predictions >= threshold are labeled transription stop site (default: 0.5"
- [discussion] 100-base bin prediction and stop-site annotation resolution: "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
