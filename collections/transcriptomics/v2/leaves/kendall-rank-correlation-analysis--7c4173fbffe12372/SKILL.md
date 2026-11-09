---
name: kendall-rank-correlation-analysis
description: Use when comparing per-cell pathogen (e.g. HIV or SARS-CoV-2) UMI counts
  from scPathoQuant's bbmap or bowtie2 aligners against CellRanger integrated-genome
  counts to assess cross-method agreement on 10x scRNA-seq data.
when_to_use_negative:
- When the input is not 10x scRNA-seq cellranger output with per-cell UMI/barcode
  tables — the correlation is defined on per-cell pathogen UMI counts, not bulk or
  gene-level expression matrices.
- When no CellRanger host+pathogen integrated-genome run exists as a baseline — there
  is no reference per-cell UMI vector to correlate against.
- For measuring read-level or phylogenetic agreement of mapped pathogen reads (use
  pathogen_al_mapped.fq.gz or IGV visualization of pathogen_al_mapped_sort.bam instead).
edam_topics:
- http://edamontology.org/topic_0202
tools:
- name: scPathoQuant
  role: Produces per-cell pathogen UMI counts (pathogen_al_counts.csv) via bbmap or
    bowtie2 remapping of host-unaligned reads; these counts are one side of each Kendall
    correlation
  repo: https://github.com/galelab/scPathoQuant
- name: cellranger
  role: Generates the 10x output folders consumed by scPathoQuant and, when run against
    the host+pathogen integrated reference, supplies the baseline per-cell pathogen
    UMI counts to correlate against
- name: bbmap
  role: Default scPathoQuant aligner whose per-cell pathogen UMI counts are correlated
    with CellRanger
- name: bowtie2
  role: Alternative scPathoQuant aligner (--very-sensitive --non-deterministic) whose
    per-cell counts are independently correlated with CellRanger
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:76182d89ae2314dd0cfe071907a31a956a10fcf0092734bae53d0f1c4f9896c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - bbmap
  - bowtie2
  - cellranger
  - scPathoQuant
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0202
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# kendall-rank-correlation-analysis

## Summary

A validation analysis that computes the Kendall rank correlation between per-cell pathogen UMI counts produced by two quantification strategies — scPathoQuant (SPQ) bbmap or bowtie2 remapping versus CellRanger with a pathogen-integrated host genome — to confirm that alternative alignment pipelines yield concordant single-cell pathogen quantification.

## When to use

Use when comparing per-cell pathogen (e.g. HIV or SARS-CoV-2) UMI counts from scPathoQuant's bbmap or bowtie2 aligners against CellRanger integrated-genome counts to assess cross-method agreement on 10x scRNA-seq data.

## When NOT to use

- When the input is not 10x scRNA-seq cellranger output with per-cell UMI/barcode tables — the correlation is defined on per-cell pathogen UMI counts, not bulk or gene-level expression matrices.
- When no CellRanger host+pathogen integrated-genome run exists as a baseline — there is no reference per-cell UMI vector to correlate against.
- For measuring read-level or phylogenetic agreement of mapped pathogen reads (use pathogen_al_mapped.fq.gz or IGV visualization of pathogen_al_mapped_sort.bam instead).

## Inputs

- 10x cellranger count output folders for each sample (containing possorted_genome_bam.bam with unaligned reads and filtered_feature_bc_matrix/raw_feature_bc_matrix)
- Pathogen reference genome fasta and optional gtf (e.g. HIV genome files or SARS-CoV-2 GCF_009858895.2 genome+gtf)
- pathogen_al_counts.csv from each scPathoQuant run (bbmap and bowtie2)
- CellRanger per-cell pathogen UMI counts from the host+pathogen integrated-genome run (sars2_cell_counts.csv-style cross-tabulation)

## Outputs

- Kendall rank correlation values of per-cell pathogen UMI counts: SPQ bbmap vs CellRanger and SPQ bowtie2 vs CellRanger, per sample/donor
- Cross-tabulation of pathogen-positive cell calls (e.g. EGFP+HIV+, EGFP-HIV+) across the three alignment methods with overlap fractions
- Per-cell pathogen count tables from pathogen_al_counts.csv for each aligner

## How to apply

First run cellranger count on each 10x sample twice: once against the host-only reference (GRCh38) so scPathoQuant can remap host-unaligned reads, and once against the combined host+pathogen reference (e.g. GRCh38+EGFP+HIV, or GRCh38+SARS-CoV-2 with a simplified fasta header such as > SARS-CoV-2) to obtain CellRanger-based pathogen UMI calls. Then run scPathoQuant on the host-only 10x outputs with the default bbmap aligner (-p 8, -p2genome pointing to the pathogen fasta/gtf folder), and repeat with -align bowtie2 --bowtie2_params="--very-sensitive --non-deterministic". Extract per-cell pathogen counts from pathogen_al_counts.csv of each run and from the CellRanger integrated-genome calls, define pathogen-positive cells per method, and compute the Kendall rank correlation of per-cell UMI counts between each SPQ aligner and CellRanger per sample/donor. Judge concordance by both the Kendall correlation magnitude and whether the positive-cell sets overlap perfectly (the paper reports 100% overlap across the three methods for both HIV donor samples and the HBEC 1-3 dpi time points).

## Related tools

- **scPathoQuant** (Produces per-cell pathogen UMI counts (pathogen_al_counts.csv) via bbmap or bowtie2 remapping of host-unaligned reads; these counts are one side of each Kendall correlation) — https://github.com/galelab/scPathoQuant
- **cellranger** (Generates the 10x output folders consumed by scPathoQuant and, when run against the host+pathogen integrated reference, supplies the baseline per-cell pathogen UMI counts to correlate against)
- **bbmap** (Default scPathoQuant aligner whose per-cell pathogen UMI counts are correlated with CellRanger)
- **bowtie2** (Alternative scPathoQuant aligner (--very-sensitive --non-deterministic) whose per-cell counts are independently correlated with CellRanger)

## Examples

```
Rscript 4.SC_Analysis.r  # run after 3.scPathoQuant.sh to compute cross-method overlaps and Kendall rank correlations of per-cell pathogen UMI counts vs CellRanger
```

## Evaluation signals

- Kendall correlations between SPQ and CellRanger per-cell pathogen UMI counts are reported for every sample/donor (e.g. written to sars2_cell_counts.csv) and are high, with SPQ bbmap vs CellRanger slightly higher than bowtie2 vs CellRanger.
- Positive-cell sets agree across all three methods: 100% overlap of EGFP+HIV+ cells for the Bradley donor samples (3653 EGFP+HIV+ and 8133 EGFP-HIV+ in donor 1; 183 and 712 in donor 2).
- SARS-CoV-2-positive cell counts match identically across CellRanger, SPQ bbmap, and SPQ bowtie2: 4,279 at 1 dpi, 13,998 at 2 dpi, 26,935 at 3 dpi.
- Per-cell count extraction is complete: pathogen_al_counts.csv exists for each aligner run and contains total pathogen reads per cell for every barcode in the 10x matrix.
- Cross-tabulation of cell classes (EGFP+HIV+, EGFP-HIV+, etc.) against CellRanger integrated-genome calls is computed for each donor before correlations are calculated.

## Limitations

- Kendall correlation of UMI counts only quantifies concordance of per-cell pathogen quantification; cell-level identity agreement must be checked separately via overlap of positive-cell sets.
- The comparison depends on a CellRanger run with a computationally integrated pathogen genome, which requires building a combined host+pathogen reference and simplifying the fasta header (e.g. > SARS-CoV-2) since it is used for total pathogen read quantification.
- The scPathoQuant temporary directory (_tmp/) files can be large and are only removed if --tmp_removal is specified.

## Evidence

- [other] the UMI counts in these cells were highly correlated with CellRanger counts for both SPQ aligners, with slightly higher correlation between SPQ bbmap and CellRanger: [evidence span withheld — anchor status: not_found]
- [other] Compute the fraction of EGFP+HIV+ cells classified consistently across the three methods and the Kendall rank correlation of per-cell HIV UMI counts versus CellRanger for each donor: [evidence span withheld — anchor status: not_found]
- [readme] This software integrates pathogen UMI counts and pathogen UMI gene counts into 10x files generated by first running: "This software integrates pathogen UMI counts and pathogen UMI gene counts into 10x files generated by first running"
- [readme] pathogen_al_counts.csv - total number of reads mapping to the pathogen in each cell: "pathogen_al_counts.csv - total number of reads mapping to the pathogen in each cell"
