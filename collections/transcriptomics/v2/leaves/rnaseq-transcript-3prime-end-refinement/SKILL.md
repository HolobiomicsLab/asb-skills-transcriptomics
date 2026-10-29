---
name: rnaseq-transcript-3prime-end-refinement
description: Use when a sorted, genome-aligned RNA-Seq BAM (e.g. HISAT2 + SAMtools)
  and a StringTie-reconstructed transcriptome (GTF) are available and you need de
  novo polyA site prediction and 3' end refinement without direct 3'-end sequencing.
when_to_use_negative:
- 'Goal is detecting coding (upstream-exon) alternative polyadenylation: aptardi evaluates
  only 3'' terminal exons, eliminating the possibility of identifying coding APA.'
- The pre-built 100-base-bin model must be reused but predictions at finer resolution
  than 100 bases are required, since aptardi annotates the 3'-most base of a positive
  100-base bin.
- 'The analysis targets taxa outside mammals: the manually engineered DNA sequence
  features may not apply outside of mammals.'
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: aptardi
  role: Machine-learning prediction of expressed polyA sites and transcript 3'-end
    refinement from RNA-Seq coverage plus DNA sequence; outputs a modified GTF/GFF
    transcriptome
  repo: https://github.com/luskry/aptardi
- name: SAMtools
  role: Sorting the HISAT2-aligned RNA-Seq BAM supplied to aptardi; must NOT be installed
    via bioconda in the same conda environment as aptardi
- name: BEDtools
  role: Dependency invoked from within the aptardi conda environment; bioconda bedtools
    must not be co-installed in that environment
- name: StringTie
  role: Reconstructs the input transcriptome GTF from the sorted BAM prior to aptardi
    3'-end refinement
- name: HISAT2
  role: Aligns RNA-Seq reads to the genome (e.g. --rna-strandness RF --dta) producing
    the BAM input for aptardi
- name: RSEM
  role: Quantitates the aptardi-modified transcriptome, and re-quantitates the high-quality
    filtered transcriptome for downstream differential expression
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:3b5185769f48b0ac1ecc735adad1aab8548e172b28e75fd14ec1e30d7ae97398
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# rnaseq-transcript-3prime-end-refinement

## Summary

Refines transcript 3'-ends and identifies expressed polyadenylation (polyA) sites in a sample-specific transcriptome using aptardi, a supervised machine-learning approach that combines RNA-Seq read coverage with DNA sequence. Outputs a modified GTF/GFF that adds aptardi transcripts on top of the unmodified input transcriptome.

## When to use

Use when a sorted, genome-aligned RNA-Seq BAM (e.g. HISAT2 + SAMtools) and a StringTie-reconstructed transcriptome (GTF) are available and you need de novo polyA site prediction and 3' end refinement without direct 3'-end sequencing.

## When NOT to use

- Goal is detecting coding (upstream-exon) alternative polyadenylation: aptardi evaluates only 3' terminal exons, eliminating the possibility of identifying coding APA.
- The pre-built 100-base-bin model must be reused but predictions at finer resolution than 100 bases are required, since aptardi annotates the 3'-most base of a positive 100-base bin.
- The analysis targets taxa outside mammals: the manually engineered DNA sequence features may not apply outside of mammals.

## Inputs

- Sorted BAM of genome-aligned RNA-Seq reads (e.g. HISAT2-aligned, SAMtools-sorted)
- Genome FASTA file with chromosome headers (e.g. hg38.fa, unzipped)
- Input transcriptome in GTF/GFF format (StringTie output stringtie.gtf or stdin from a pipe)
- Pre-built model file model.hdf5 (Mode 1)
- Pre-built scale file scale.pk (Mode 1)

## Outputs

- New GTF/GFF file (name_aptardi.gtf) containing original input transcripts plus added aptardi transcripts with refined 3'-ends
- Optionally, a custom trained model (.hdf5) and scale file from Mode 2 using gold-standard polyA sites

## How to apply

Run aptardi in Mode 1 with a sorted BAM, an unzipped genome FASTA (e.g. hg38.fa), and an input GTF (designed for StringTie output), supplying the pre-built HBR model.hdf5 and scale.pk from src/aptardi/ml_scale. Keep the default bin size --w 100 (the pre-built model was trained on 100-base bins and should not be altered), maximum analyzed length --i 300 bins (= 30,000 bases), probability threshold --p 0.5 for labeling transcription stop sites, and mate orientation --a fr for firststrand Illumina paired-end libraries. Aptardi bins the modified 3'-terminal exons of input transcript models into 100-base increments and computes 27 features per bin (14 RNA-Seq, 12 DNA-sequence, 1 transcriptome-derived), then annotates each positive-prediction bin's 3'-most base as the transcript stop site. It never evaluates intron junctions and removes no transcripts from the input transcriptome, so downstream, quantitate with RSEM and apply the conservative filter of removing transcripts without at least 1 count in 2/3 of samples to reduce redundant aptardi transcripts, then re-quantitate for differential expression.

## Related tools

- **aptardi** (Machine-learning prediction of expressed polyA sites and transcript 3'-end refinement from RNA-Seq coverage plus DNA sequence; outputs a modified GTF/GFF transcriptome) — https://github.com/luskry/aptardi
- **SAMtools** (Sorting the HISAT2-aligned RNA-Seq BAM supplied to aptardi; must NOT be installed via bioconda in the same conda environment as aptardi)
- **BEDtools** (Dependency invoked from within the aptardi conda environment; bioconda bedtools must not be co-installed in that environment)
- **StringTie** (Reconstructs the input transcriptome GTF from the sorted BAM prior to aptardi 3'-end refinement)
- **HISAT2** (Aligns RNA-Seq reads to the genome (e.g. --rna-strandness RF --dta) producing the BAM input for aptardi)
- **RSEM** (Quantitates the aptardi-modified transcriptome, and re-quantitates the high-quality filtered transcriptome for downstream differential expression)

## Examples

```
aptardi --b sorted.bam --f hg38.fa --r stringtie.gtf --n model.hdf5 --t scale.pk --g name_aptardi.gtf --o output_dir --w 100 --i 300 --p 0.5 --a fr
```

## Evaluation signals

- name_aptardi.gtf exists, parses as valid GTF/GFF, and contains both the original input transcripts and added aptardi transcripts (aptardi removes no transcripts from the input transcriptome).
- Predicted 3' ends are annotated as the 3'-most base of positive-prediction 100-base bins, and predictions are called at the default 0.5 probability threshold.
- The pipeline evaluated only the 3' terminal exons of input transcripts — no intron junctions and no upstream exons were assessed.
- Post-filtering, transcripts without at least 1 count in 2/3 of samples are removed, and the filtered high-quality transcriptome is re-quantitated with RSEM.
- Environment verification: aptardi --version, samtools --version, and bedtools --version all run, with bioconda samtools/bedtools NOT installed in the same conda environment as aptardi.

## Limitations

- aptardi makes predictions on 100 base bins (default --w 100); this resolution is limited because polyA sites wiggle up to 30 nucleotides, and altering --w with the pre-built model is not recommended.
- aptardi does not distinguish transcripts overlapping the same genomic region, so it may add a single polyA site to multiple transcripts per gene, or attribute a stop site to the wrong transcript; mitigate by curating a more-selective input transcriptome and filtering transcripts without at least 1 count in 2/3 of samples.
- Upstream exons are not analyzed, eliminating the possibility of identifying coding APA.
- Performance can degrade with unstranded RNA-Seq or different species (modestly worse on BNLx and SHR rat data sets), and detection depends on sequencing depth (modest polyA site counts in low-depth mouse tissues).
- The bioconda samtools and bedtools packages may prevent aptardi from working if installed in the same conda environment.

## Evidence

- [abstract] aptardi leverages both DNA sequence and RNA sequencing in a machine learning framework to predict polyA sites: [evidence span withheld — anchor status: not_found]
- [methods] 27 features per 100-base bin: 14 RNA-Seq, 12 DNA sequence, 1 transcriptome-derived: [evidence span withheld — anchor status: not_found]
- [readme] Aptardi evaluates only 3' terminal exons and does not remove input transcripts: "Note that aptardi does not evaluate intron junctions but rather only evalutes 3' terminal exons of input transcripts and annotates 3' ends accordingly."
- [readme] Conservative filter: remove transcripts without at least 1 count in 2/3 of samples: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
- [discussion] 100-base bin resolution limitation and 3'-most base annotation: "aptardi makes predictions on 100 base bins (for positive predictions, it annotates the transcript stop site as the 3′ most base in the given 100 base bin)"
