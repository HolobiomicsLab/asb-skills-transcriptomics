---
name: polyadenylation-informed-transcriptome-analysis
description: Use when you need to test whether adding aptardi-predicted polyA transcripts
  to a GTF changes which transcripts are called differentially expressed between two
  conditions (e.g., BNLx vs SHR rat brain).
when_to_use_negative:
- You only need polyA site prediction itself, not downstream quantitation/differential
  expression against the modified transcriptome
- Your input transcriptome is a feature/count matrix already, with no GTF to modify
  or re-quantitate
- You require detection of coding alternative polyadenylation in upstream exons —
  aptardi only evaluates 3' terminal exons, eliminating coding APA detection
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: aptardi
  role: Predicts expressed polyadenylation sites from RNA-Seq coverage plus DNA sequence
    and outputs a modified GTF with refined 3'-ends used as the analysis transcriptome
  repo: github.com/luskry/aptardi
- name: RSEM
  role: Quantitates samples against both the aptardi-modified and original transcriptomes;
    re-quantitation after filtering feeds DESeq2
- name: DESeq2
  role: Performs differential expression testing (BNLx vs SHR) on filtered RSEM quantitations;
    transcripts with p <= 0.001 are extracted and compared across transcriptomes
- name: StringTie
  role: Reconstructs the initial transcriptome GTF from sorted BAMs that serves as
    aptardi's input transcriptome
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:cf7dc54a85fdfa712e5083d3c609acf12b1e30f8f78683eb41154408ff48ab09
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - RSEM
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# polyadenylation-informed-transcriptome-analysis

## Summary

Quantify and differentially express samples against an aptardi-modified transcriptome that adds sample-specific polyadenylation sites, comparing results to the original StringTie/Ensembl transcriptome using RSEM and DESeq2.

## When to use

Use when you need to test whether adding aptardi-predicted polyA transcripts to a GTF changes which transcripts are called differentially expressed between two conditions (e.g., BNLx vs SHR rat brain).

## When NOT to use

- You only need polyA site prediction itself, not downstream quantitation/differential expression against the modified transcriptome
- Your input transcriptome is a feature/count matrix already, with no GTF to modify or re-quantitate
- You require detection of coding alternative polyadenylation in upstream exons — aptardi only evaluates 3' terminal exons, eliminating coding APA detection

## Inputs

- Sorted BAM files of genome-aligned RNA-Seq reads (e.g., HISAT2-aligned, SAMtools-sorted BNLx/SHR rat brain samples)
- Original transcriptome GTF (StringTie reconstruction and/or Ensembl annotation)
- Genome FASTA file (e.g., hg38.fa or rat genome) with chromosome headers
- Pre-built aptardi model (model.hdf5) and scale file (scale.pk), or gold-standard polyA sites for Mode 2 custom model building
- Sample metadata/design table for DESeq2 contrast definition

## Outputs

- Aptardi-modified transcriptome GTF incorporating aptardi transcripts alongside original transcripts
- RSEM quantitation matrices for aptardi and original transcriptomes (before and after filtering)
- High-quality filtered transcriptome (transcripts with >=1 count in 2/3 of samples)
- DESeq2 differential expression results per transcriptome with p-values (p <= 0.001 set), e.g., differential_expression_results.tsv
- List of aptardi-added differentially expressed transcripts (present in aptardi DE set, absent from original DE set)

## How to apply

Run aptardi on genome-aligned sorted BAMs plus an initial GTF (StringTie/Ensembl) and genome FASTA to obtain a modified transcriptome GTF; aptardi only refines 3' terminal exons and never removes input transcripts. Quantitate both the aptardi-modified and original transcriptomes with RSEM across all samples. Apply the paper's conservative quality filter — remove transcripts without at least 1 count in 2/3 of samples — then re-quantitate the filtered high-quality transcriptomes with RSEM. Run DESeq2 differential expression testing (e.g., BNLx vs SHR) on both filtered sets and extract transcripts with p <= 0.001; identify aptardi-added DE transcripts as those in the aptardi set but absent from the original set. The rationale is that aptardi's added 3'-ends increase captured true polyA sites, which changed DE counts from 918/28,329 (original) to 1166/32,348 (aptardi) in the published BNLx/SHR brain analysis.

## Related tools

- **aptardi** (Predicts expressed polyadenylation sites from RNA-Seq coverage plus DNA sequence and outputs a modified GTF with refined 3'-ends used as the analysis transcriptome) — github.com/luskry/aptardi
- **RSEM** (Quantitates samples against both the aptardi-modified and original transcriptomes; re-quantitation after filtering feeds DESeq2)
- **DESeq2** (Performs differential expression testing (BNLx vs SHR) on filtered RSEM quantitations; transcripts with p <= 0.001 are extracted and compared across transcriptomes)
- **StringTie** (Reconstructs the initial transcriptome GTF from sorted BAMs that serves as aptardi's input transcriptome)

## Examples

```
conda create -n aptardi_env --channel conda-forge -c bioconda aptardi; aptardi --o out_dir --f genome.fa --r stringtie.gtf --b sorted.bam --n model.hdf5 --t scale.pk --g aptardi.gtf
```

## Evaluation signals

- Number of transcripts passing p <= 0.001 matches reported values: 1166 of 32,348 (aptardi transcriptome) and 918 of 28,329 (original transcriptome)
- Filtered transcriptome contains only transcripts with at least 1 count in 2/3 of samples
- Aptardi output GTF is a superset: it contains all original input transcripts plus added aptardi transcripts (aptardi never removes transcripts)
- Aptardi-added DE transcripts are disjoint from the original transcriptome's DE set by construction (ID comparison)
- All results (DE totals and aptardi-added DE list) are written to differential_expression_results.tsv

## Limitations

- aptardi makes predictions on 100 base bins, so annotated stop sites can wiggle up to ~30 nucleotides and resolution is limited
- aptardi does not distinguish transcripts overlapping the same genomic region, so a single 3'-end can be incorporated into multiple transcripts per gene, possibly attributing a stop site to the wrong transcript
- Upstream exons are not analyzed, effectively eliminating the possibility of identifying coding APA; redundant/false-positive aptardi transcripts must be filtered, especially when the input GTF has many isoforms sharing 3' terminal exons
- Performance was modestly worse on the BNLx and SHR data sets, possibly due to unstranded RNA-Seq and/or species differences from the training data (true polyA sites derived from Sprague Dawley rat)

## Evidence

- [full_text] Differential expression analysis of the BNLx and SHR rat brain data sets yielded more significant transcripts (p value ≤ 0.001) with the aptardi modified transcriptome (1166 of 32,348) than with the original transcriptome (918 of 28,329).: [evidence span withheld — anchor status: not_found]
- [readme] Filtering recommendation from README: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
- [readme] Filtering pipeline summary: "1. Quantitate aptardi transcriptome with RSEM 2. Generate high quality transcriptome by removing transcripts without at least 1 count in 2/3 of samples 3. Proceed with downstream analyses (we"
- [readme] Output GTF is superset: "Aptardi analyzes the input gtf file and outputs a new gtf file with aptardi transcripts added along with original transcripts, i.e. aptardi does not remove any transcripts from the input"
- [readme] Refinement scope limitation: "Note that aptardi does not evaluate intron junctions but rather only evalutes 3' terminal exons of input transcripts and annotates 3' ends accordingly."
