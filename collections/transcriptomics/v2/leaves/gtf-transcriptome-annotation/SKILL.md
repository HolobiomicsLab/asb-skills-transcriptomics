---
name: gtf-transcriptome-annotation
description: Use when a sample-specific GTF/GFF transcriptome (e.g., StringTie output)
  needs refined 3' ends / expressed polyadenylation sites before quantitation or differential
  expression analysis.
when_to_use_negative:
- The goal is to detect coding (upstream-exon) alternative polyadenylation — aptardi
  only evaluates 3' terminal exons, so upstream APA is out of scope
- The input transcriptome has many isoforms sharing 3' terminal exons and no post-hoc
  filtering is planned — redundant false-positive aptardi transcripts will be enumerated
- The task requires resolving which overlapping transcript isoform truly owns a polyA
  site — aptardi does not distinguish transcripts overlapping the same genomic region
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0622
tools:
- name: aptardi
  role: Machine-learning prediction of 3' ends / polyA sites from RNA-Seq and DNA
    sequence, outputting an augmented GTF/GFF transcriptome
  repo: https://github.com/luskry/aptardi
- name: StringTie
  role: Reconstructs the input transcriptome (stringtie.gtf) from the sorted BAM that
    aptardi refines
- name: SAMtools
  role: Sorts the HISAT2-aligned BAM used as aptardi's --b input; must not be installed
    via bioconda in the aptardi environment
- name: BEDtools
  role: Dependency callable from the aptardi conda environment; must not be installed
    via bioconda in the same environment
- name: HISAT2
  role: Aligns RNA-Seq reads to the genome before aptardi prediction
- name: RSEM
  role: Quantitates the aptardi transcriptome for the recommended filtering workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:beb50839db9769f55d0c3b2efa9db4fbcad92c156b9c623c80c4dcdd2de0f17e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - BEDtools
  - HISAT2
  - RSEM
  - SAMtools
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# gtf-transcriptome-annotation

## Summary

Construct a sample-specific transcriptome annotation by running aptardi, which evaluates the 3' terminal exons of input GTF/GFF transcripts with RNA-Seq coverage and DNA sequence via a machine-learning model and outputs a new GTF with aptardi transcripts added on top of the unmodified input transcriptome.

## When to use

Use when a sample-specific GTF/GFF transcriptome (e.g., StringTie output) needs refined 3' ends / expressed polyadenylation sites before quantitation or differential expression analysis.

## When NOT to use

- The goal is to detect coding (upstream-exon) alternative polyadenylation — aptardi only evaluates 3' terminal exons, so upstream APA is out of scope
- The input transcriptome has many isoforms sharing 3' terminal exons and no post-hoc filtering is planned — redundant false-positive aptardi transcripts will be enumerated
- The task requires resolving which overlapping transcript isoform truly owns a polyA site — aptardi does not distinguish transcripts overlapping the same genomic region

## Inputs

- Sorted BAM of HISAT2-aligned RNA-Seq reads (e.g., sorted.bam, --a fr for firststrand Illumina libraries)
- Reference genome FASTA (e.g., hg38.fa, unzipped, chromosome-named headers)
- Input transcriptome in GTF/GFF format (e.g., stringtie.gtf from StringTie reconstruction)
- Pre-built model file model.hdf5 and scale file scale.pk from src/aptardi/ml_scale (HBR-trained)

## Outputs

- Modified sample-specific transcriptome GTF (name_aptardi.gtf) containing original input transcripts plus added aptardi transcripts
- Optionally, a high-quality filtered transcriptome after RSEM quantitation and removal of transcripts with <1 count in 2/3 of samples

## How to apply

Align RNA-Seq reads with HISAT2, sort with SAMtools, and reconstruct a transcriptome with StringTie (e.g., 'stringtie sorted.bam --rf -o stringtie.gtf'), then run aptardi in Mode 1 with 'aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --n model.hdf5 --t scale.pk --g name_aptardi.gtf --o output_dir', keeping the canonical defaults: bin size --w 100 (not to be altered with the pre-built HBR model), maximum analyzed length --i 300 bins (= 30,000 bases), probability threshold --p 0.5 for labeling transcription stop sites, and --a fr mate orientation for firststrand Illumina libraries. Install via 'conda create -n aptardi_env --channel conda-forge -c bioconda aptardi' and ensure bioconda samtools and bedtools are NOT installed in the same environment. Aptardi bins the modified 3'-terminal exons into 100-base increments, computes 27 features per bin (14 RNA-Seq, 12 DNA sequence, 1 transcriptome-derived), and annotates predicted 3' ends as the 3'-most base of positive-prediction bins; it does not evaluate intron junctions and removes no input transcripts. Because aptardi identifies genomic regions and may enumerate one polyA site across multiple overlapping isoforms, quantitate with RSEM and filter out transcripts without at least 1 count in 2/3 of samples to generate a high-quality transcriptome, then re-quantitate with RSEM.

## Related tools

- **aptardi** (Machine-learning prediction of 3' ends / polyA sites from RNA-Seq and DNA sequence, outputting an augmented GTF/GFF transcriptome) — https://github.com/luskry/aptardi
- **StringTie** (Reconstructs the input transcriptome (stringtie.gtf) from the sorted BAM that aptardi refines)
- **SAMtools** (Sorts the HISAT2-aligned BAM used as aptardi's --b input; must not be installed via bioconda in the aptardi environment)
- **BEDtools** (Dependency callable from the aptardi conda environment; must not be installed via bioconda in the same environment)
- **HISAT2** (Aligns RNA-Seq reads to the genome before aptardi prediction)
- **RSEM** (Quantitates the aptardi transcriptome for the recommended filtering workflow)

## Examples

```
aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --n model.hdf5 --t scale.pk --g name_aptardi.gtf --o output_dir
```

## Evaluation signals

- name_aptardi.gtf exists, parses as GTF/GFF, and contains both the original input transcripts and added aptardi transcripts (aptardi removes no transcripts from the input transcriptome)
- Predicted 3' ends are annotated as the 3'-most base of 100-base bins whose prediction probability meets the default --p 0.5 threshold
- Only the 3' terminal exons of input transcripts were evaluated — aptardi does not assess intron junctions
- Non-redundancy check: after RSEM quantitation and filtering transcripts without at least 1 count in 2/3 of samples, redundant aptardi transcripts enumerated for overlapping isoforms are removed
- Verify model files used are the pre-built HBR model (model.hdf5, scale.pk) with default bin size --w 100, since the pre-built model was trained on 100-base bins

## Limitations

- Aptardi makes predictions on 100 base bins (resolution limitation), though polyA sites can wiggle up to 30 nucleotides and few bins contained multiple sites
- Aptardi does not distinguish transcripts overlapping the same genomic region, so a single 3' end may be incorporated into multiple transcripts per gene, possibly attributing a stop site to the wrong transcript
- Upstream exons are not analyzed, effectively eliminating the possibility of identifying coding APA; manually engineered DNA sequence features may not apply outside mammals
- If the input transcriptome has many isoforms sharing 3' terminal exons, aptardi will enumerate redundant (false positive) transcripts, making post-hoc filtering highly recommended

## Evidence

- [readme] aptardi combines RNA-Seq data from the sample with the genome (DNA sequence) of the sample to identify 3' ends of transcripts using machine learning: "combines RNA-Seq data from the sample with the genome (DNA sequence) of the sample to identify 3' ends of transcripts using machine learning"
- [readme] Aptardi outputs original transcripts plus aptardi transcripts: "outputs a new gtf file with aptardi transcripts added along with original transcripts, i.e. aptardi does not remove any transcripts from the input transcriptome"
- [readme] Only 3' terminal exons are evaluated: "aptardi does not evaluate intron junctions but rather only evalutes 3' terminal exons of input transcripts and annotates 3' ends accordingly"
- [methods] Default parameters for Mode 1 prediction: "--p <float>			Probability threshold, predictions >= threshold are labeled transription stop site (default: 0.5"
- [other] Bin-level feature computation: [evidence span withheld — anchor status: not_found]
- [readme] Post-annotation filtering recommendation: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
