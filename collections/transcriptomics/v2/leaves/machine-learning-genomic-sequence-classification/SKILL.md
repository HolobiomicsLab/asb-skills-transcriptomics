---
name: machine-learning-genomic-sequence-classification
description: Use when you need to identify expressed, sample-specific polyadenylation
  sites from genome-aligned RNA-Seq (sorted BAM) plus DNA sequence, especially de
  novo proximal APA isoforms that standard assemblers like StringTie miss at 3' terminal
  exons.
when_to_use_negative:
- You need to discover coding APA events in upstream exons — aptardi only analyzes
  3' terminal exons of input transcripts
- Your data lack polyA site relevance or come from non-mammalian taxa — the manually
  engineered DNA sequence features may not apply outside mammals
- You only have an a priori annotated set of polyA sites and no sample RNA-Seq coverage
  — tools like MISO/QAPA requiring known sites, not aptardi's ML prediction, fit that
  task
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: aptardi
  role: Machine-learning classifier that combines DNA sequence and RNA-Seq coverage
    in 100-base bins to predict expressed polyA sites and output a modified transcriptome
    GTF
  repo: github.com/luskry/aptardi
- name: StringTie
  role: Reconstructs the sample transcriptome (stringtie.gtf with --rf -G <guide_file>)
    that serves as the initial transcriptome input to aptardi
- name: HISAT2
  role: Aligns strand-specific paired-end RNA-Seq reads (--rna-strandness RF --dta)
    to the genome to produce the sorted BAM aptardi consumes
- name: SAMtools
  role: Sorts the HISAT2 alignments into the sorted.bam required by aptardi (v1.9)
- name: RSEM
  role: Quantitates the aptardi transcriptome so the 1-count-in-2/3-of-samples filter
    can remove redundant aptardi transcripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:e7f0c113d1ed84e7e258ab42e431ae4202bd23ff5fb52669c82be068ac5f4589
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HISAT2
  - RSEM
  - SAMtools
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# machine-learning-genomic-sequence-classification

## Summary

A supervised machine-learning approach (aptardi) that classifies genomic 100-base bins as transcription stop sites by combining DNA nucleotide sequence features with sample-specific RNA-Seq coverage, refining transcript 3'-ends into a sample-specific GTF transcriptome.

## When to use

Use when you need to identify expressed, sample-specific polyadenylation sites from genome-aligned RNA-Seq (sorted BAM) plus DNA sequence, especially de novo proximal APA isoforms that standard assemblers like StringTie miss at 3' terminal exons.

## When NOT to use

- You need to discover coding APA events in upstream exons — aptardi only analyzes 3' terminal exons of input transcripts
- Your data lack polyA site relevance or come from non-mammalian taxa — the manually engineered DNA sequence features may not apply outside mammals
- You only have an a priori annotated set of polyA sites and no sample RNA-Seq coverage — tools like MISO/QAPA requiring known sites, not aptardi's ML prediction, fit that task

## Inputs

- Sorted BAM of genome-aligned RNA-Seq reads (e.g., HISAT2 output)
- Genome FASTA with chromosome headers (e.g., hg38.fa)
- Input transcriptome GTF/GFF, typically StringTie output (stringtie.gtf)
- Pre-trained model file (model.hdf5 from src/aptardi/ml_scale)
- Pre-trained scale file (scale.pk from src/aptardi/ml_scale)

## Outputs

- Modified sample-specific transcriptome GTF (name_aptardi.gtf) containing aptardi-annotated transcripts plus original input transcripts
- Refined transcript 3'-end / polyA site coordinates for the sample

## How to apply

Align strand-specific paired-end reads with HISAT2 (--rna-strandness RF --dta), sort with SAMtools, and reconstruct the transcriptome with StringTie (--rf -G <guide_file>) to produce the input stringtie.gtf. Run aptardi in Mode 1 with the sorted BAM, a chromosome-headered genome FASTA (e.g., hg38.fa), the StringTie GTF, and the pre-built model.hdf5 and scale.pk from src/aptardi/ml_scale. Keep the default 100-base prediction bins (do not change --w with the pre-built model), the default probability threshold 0.5 via --p, and the paired-end mate orientation --a fr for firststrand Illumina libraries. aptardi evaluates only 3' terminal exons of input transcripts (not intron junctions), classifying bins where predictions >= 0.5 as transcription stop sites and annotating the 3'-most base of the positive bin. Output name_aptardi.gtf adds aptardi transcripts to the original ones (nothing is removed); afterwards quantitate with RSEM and apply the conservative filter removing transcripts without at least 1 count in 2/3 of samples to remove redundant false-positive aptardi transcripts.

## Related tools

- **aptardi** (Machine-learning classifier that combines DNA sequence and RNA-Seq coverage in 100-base bins to predict expressed polyA sites and output a modified transcriptome GTF) — github.com/luskry/aptardi
- **StringTie** (Reconstructs the sample transcriptome (stringtie.gtf with --rf -G <guide_file>) that serves as the initial transcriptome input to aptardi)
- **HISAT2** (Aligns strand-specific paired-end RNA-Seq reads (--rna-strandness RF --dta) to the genome to produce the sorted BAM aptardi consumes)
- **SAMtools** (Sorts the HISAT2 alignments into the sorted.bam required by aptardi (v1.9))
- **RSEM** (Quantitates the aptardi transcriptome so the 1-count-in-2/3-of-samples filter can remove redundant aptardi transcripts)

## Examples

```
aptardi --o <outdir> --f hg38.fa --r stringtie.gtf --b sorted.bam -n model.hdf5 -t scale.pk --a fr -g sample_aptardi.gtf
```

## Evaluation signals

- name_aptardi.gtf contains aptardi transcripts for both samples in addition to all original input transcripts (aptardi never removes transcripts)
- Predicted proximal polyA sites recover known loci, e.g., CCND1 chr11:69,651,917, DICER1 chr11:95,090,264, TIMP2 chr17:78,855,465, matching the reported CFIm25-knockdown positions
- The aptardi model achieves average precision roughly twice that of a standard transcriptome assembler and over three-fold improved recall on gold-standard 3'-sequencing labels
- Filtering (RSEM quantitation, remove transcripts without ≥1 count in 2/3 of samples) reduces redundant aptardi transcripts arising from isoforms sharing 3' terminal exons
- Threshold and bin parameters remain at defaults: --p 0.5 (0,1) and --w 100 (25–200), since the pre-built model was trained on 100-base bins

## Limitations

- aptardi predicts on 100-base bins, a resolution limitation, though polyA sites wiggle up to 30 nucleotides and few bins contained multiple sites
- aptardi incorporates a single 3'-end into multiple transcripts per gene because it does not distinguish transcripts overlapping the same genomic region, possibly attributing a stop site to the wrong transcript
- Upstream exons are not analyzed, eliminating the possibility of identifying coding APA
- Manually engineered DNA sequence features may not apply to taxa outside mammals
- The bioconda samtools and bedtools packages may prevent aptardi from working and should not be installed in the same conda environment

## Evidence

- [abstract] as input aptardi takes DNA nucleotide sequence, genome-aligned RNA-Seq data, and an initial transcriptome: "as input aptardi takes DNA nucleotide sequence, genome-aligned RNA-Seq data, and an initial transcriptome"
- [abstract] The average precision of the aptardi model is twice that of a standard transcriptome assembler.: "The average precision of the aptardi model is twice that of a standard transcriptome assembler."
- [discussion] aptardi makes predictions on 100 base bins: "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
- [readme] Probability threshold for stop site labeling: "Probability threshold, predictions >= threshold are labeled transription stop site (default: 0.5, value must be constrained by (0, 1))"
- [readme] Filtering to high quality aptardi transcripts: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
