---
name: alternative-polyadenylation-isoform-comparison
description: Use when paired-end RNA-Seq from two or more conditions is available
  and the goal is to detect proximal APA transcripts (shifted/shortened 3' ends) induced
  by a perturbation, such as knockdown of a polyadenylation factor like CFIm25.
when_to_use_negative:
- 'The goal is de novo polyA site prediction without an initial transcriptome: aptardi
  requires an input transcriptome (GTF/GFF, designed for StringTie output) and only
  evaluates 3'' terminal exons, not intron junctions'
- 'Only a single condition/sample is available with no comparison transcriptome: isoform-specificity
  claims from the two-sample comparison cannot be made (use a count threshold filter
  instead)'
- 'The question is coding APA within upstream exons: aptardi does not analyze upstream
  exons, so coding APA cannot be identified'
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: HISAT2
  role: Aligns control and knockdown paired-end RNA-Seq reads with --rna-strandness
    RF --dta for firststrand protocol
- name: SAMtools
  role: Sorts HISAT2 alignments into sorted.bam required as aptardi input
- name: StringTie
  role: Reconstructs each sample's transcriptome (stringtie.gtf) from sorted.bam using
    Ensembl guide annotation with --rf, as the input transcriptome for aptardi
- name: aptardi
  role: Predicts sample-specific polyA sites (Mode 1 with pre-built model.hdf5/scale.pk)
    and outputs name_aptardi.gtf whose 3' ends are compared between conditions to
    identify proximal APA isoforms
  repo: github.com/luskry/aptardi
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:c24d77aec73c7ab7c938813cd563d3e61d54e8d0ae169e497638fe1a211d6f56
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HISAT2
  - SAMtools
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# alternative-polyadenylation-isoform-comparison

## Summary

Identifies sample-specific alternative polyadenylation (APA) transcript isoforms by building sample-specific transcriptomes with aptardi (Mode 1, pre-built model) from HISAT2/SAMtools/StringTie outputs and comparing predicted 3' ends between conditions, e.g. control vs. CFIm25 knockdown HeLa cells.

## When to use

Use when paired-end RNA-Seq from two or more conditions is available and the goal is to detect proximal APA transcripts (shifted/shortened 3' ends) induced by a perturbation, such as knockdown of a polyadenylation factor like CFIm25.

## When NOT to use

- The goal is de novo polyA site prediction without an initial transcriptome: aptardi requires an input transcriptome (GTF/GFF, designed for StringTie output) and only evaluates 3' terminal exons, not intron junctions
- Only a single condition/sample is available with no comparison transcriptome: isoform-specificity claims from the two-sample comparison cannot be made (use a count threshold filter instead)
- The question is coding APA within upstream exons: aptardi does not analyze upstream exons, so coding APA cannot be identified

## Inputs

- Paired-end RNA-Seq FASTQ files from control and perturbed/knockdown samples (e.g. control and CFIm25-knockdown HeLa cells, firststrand Illumina protocol)
- HISAT2 genome index for the reference genome (e.g. hg38)
- Chromosome-headered genome FASTA (e.g. hg38.fa from the aptardi distribution)
- Ensembl guide annotation (GTF) for StringTie reconstruction
- Pre-built aptardi model.hdf5 and scale.pk from src/aptardi/ml_scale

## Outputs

- Per-sample sorted BAM of genome-aligned reads (sorted.bam)
- Per-sample StringTie transcript reconstruction (stringtie.gtf)
- Per-sample aptardi-modified transcriptome (name_aptardi.gtf) containing original plus aptardi transcripts
- Set of proximal APA isoforms detected only in the perturbed sample, with genomic 3'-end coordinates

## How to apply

Align control and knockdown paired-end reads with HISAT2 using --rna-strandness RF and --dta for a firststrand Illumina protocol, then sort alignments with SAMtools into sorted.bam. Reconstruct each sample's transcriptome with StringTie (sorted.bam --rf -o stringtie.gtf -G <Ensembl guide file>). Run aptardi in Mode 1 (canonical usage) with the sorted BAM, a chromosome-headered genome FASTA (e.g. hg38.fa), and the StringTie GTF, supplying the pre-built model.hdf5 and scale.pk from src/aptardi/ml_scale and keeping defaults (100-base prediction bins, probability threshold 0.5, paired-end orientation fr), saving output as name_aptardi.gtf. Finally, compare aptardi-annotated 3' ends between the control and knockdown transcriptomes to find proximal APA isoforms detected only in the knockdown sample; filter redundant aptardi transcripts (e.g. RSEM quantitation followed by removing transcripts without at least 1 count in 2/3 of samples) before asserting isoform specificity.

## Related tools

- **HISAT2** (Aligns control and knockdown paired-end RNA-Seq reads with --rna-strandness RF --dta for firststrand protocol)
- **SAMtools** (Sorts HISAT2 alignments into sorted.bam required as aptardi input)
- **StringTie** (Reconstructs each sample's transcriptome (stringtie.gtf) from sorted.bam using Ensembl guide annotation with --rf, as the input transcriptome for aptardi)
- **aptardi** (Predicts sample-specific polyA sites (Mode 1 with pre-built model.hdf5/scale.pk) and outputs name_aptardi.gtf whose 3' ends are compared between conditions to identify proximal APA isoforms) — github.com/luskry/aptardi

## Examples

```
hisat2 -q -p 5 --reorder -t --rna-strandness RF --dta -x <hisat2_index> -1 <R1.fq.gz> -2 <R2.fq.gz> | samtools sort -o sorted.bam; stringtie sorted.bam --rf -o stringtie.gtf -G <guide_file>; aptardi --o <outdir> --f hg38.fa --r stringtie.gtf --b sorted.bam -n model.hdf5 -t scale.pk -g <name>_aptardi.gtf
```

## Evaluation signals

- name_aptardi.gtf is produced for both samples and contains aptardi transcripts in addition to the original input transcripts (aptardi never removes input transcripts)
- Predicted proximal polyA sites match reported loci: CCND1 chr11:69,651,917; DICER1 chr11:95,090,264; TIMP2 chr17:78,855,465
- Proximal APA calls for CCND1, DICER1, and TIMP2 are recovered exclusively in the CFIm25-knockdown transcriptome, not the control
- Redundant aptardi transcripts are filtered (e.g. transcripts without at least 1 count in 2/3 of samples removed after RSEM quantitation) before asserting isoform-specificity
- aptardi run parameters are defaults: 100-base prediction bins (unchanged because the pre-built model was built on 100-base bins), probability threshold 0.5, paired-end orientation fr

## Limitations

- aptardi makes predictions on 100 base bins (annotating the 3'-most base in a positive bin), a resolution limitation since polyA sites can wiggle up to ~30 nucleotides
- aptardi does not distinguish transcripts overlapping the same genomic region, so a single 3'-end may be added to multiple transcripts per gene, and a stop site may be attributed to the wrong transcript isoform
- Upstream exons are not analyzed, so coding APA cannot be identified
- False-positive/redundant aptardi transcripts proliferate when the input transcriptome has many isoforms sharing 3' terminal exons; mitigate with RSEM quantitation plus a 1-count-in-2/3-of-samples filter or a more-selective input transcriptome
- Manually engineered DNA sequence features may not apply outside mammals; performance may drop for unstranded RNA-Seq or different species

## Evidence

- [other] Align the control and CFIm25-knockdown HeLa paired-end reads with HISAT2 (--rna-strandness RF, --dta, firststrand Illumina protocol) and sort the resulting alignments with SAMtools into sorted.bam.: [evidence span withheld — anchor status: not_found]
- [methods] stringtie sorted.bam --rf -o stringtie.gtf -G <guide_file>: "stringtie sorted.bam --rf -o stringtie.gtf -G <guide_file>"
- [readme] we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts: "removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
- [discussion] aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin): "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
- [discussion] aptardi incorporates a single 3′-end into multiple transcripts for each gene listed because it does not distinguish transcripts overlapping the same genomic region: "aptardi incorporates a single 3′-end into multiple transcripts for each gene listed because it does not distinguish transcripts overlapping the same genomic region"
- [discussion] Applying aptardi in control and CFIm25 knockdown RNA-Seq data demonstrated its: "Applying aptardi in control and CFIm25 knockdown RNA-Seq data demonstrated its"
