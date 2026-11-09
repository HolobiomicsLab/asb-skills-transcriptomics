---
name: cluster-size-read-fraction-statistics
description: Use when FASTQ-to-SAM UMI pipelines have produced per-position UMI cluster
  data for one or more UMI-based RNA-Seq datasets and you need to quantify what proportion
  of reads fall into size-1 clusters versus mapping-shifted clusters.
when_to_use_negative:
- Input data contains no UMIs — the article states 'In the absence of UMI this artefact
  is undetectable', so cluster-size statistics are undefined
- The goal is paired-end or splice-aware transcript-level quantification — the pipeline
  is designed for single-ended sequencing with STAR specifying no introns
- You need the true biological distribution of mapping-shift sizes — the article notes
  the 3 bp collapse threshold is arbitrary and the true distribution is unknown
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: fastq_qual_filter
  role: C program that quality-filters FASTQ reads on a substring (e.g., UMI at min_ok
    30, offset 0, length 10) before UMI clipping
  repo: https://github.com/ncgr/UMI-analysis
- name: fastq_umi_clipper
  role: C program that moves the 5' UMI into the FASTQ header and deletes protocol-derived
    trailing Gs (g_length) before alignment
  repo: https://github.com/ncgr/UMI-analysis
- name: STAR
  role: Aligns clipped reads to the genomic reference specifying no introns, producing
    the sorted SAM needed for UMI collapsing
  repo: https://github.com/alexdobin/STAR
- name: umi_counterizer.pl
  role: Perl script collapsing duplicate UMIs at the same mapping position with a
    max_edit_distance of 3 bp and counting UMIs per position
  repo: https://github.com/ncgr/UMI-analysis
- name: umi_analyzer_uthash
  role: C program using uthash to find all genomic positions where each UMI maps (umi_positions.txt),
    the key input for cluster-size pooling
  repo: https://github.com/ncgr/UMI-analysis
- name: umi_clusterizer.pl
  role: Perl script counting unique UMIs per genomic position into position_umi_counts.bed,
    the basis of the cluster-size read-fraction statistics
  repo: https://github.com/ncgr/UMI-analysis
- name: bedtools
  role: Optionally intersects position_umi_counts.bed with GFF gene annotations to
    confirm gene-level behaviour of the clusters
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:ca31c361a7d36da8baca48c1e7a201744459563b5e95fc5786f9bfabdfed18c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncgr/UMI-analysis
  tools:
  - C
  - STAR
  - Text::Levenshtein::XS
  - Try::Tiny
  - bedtools
  - fastq_qual_filter
  - fastq_umi_clipper
  - pcregrep
  - umi_analyzer_uthash
  - umi_clusterizer.pl
  - umi_counterizer.pl
  - uthash
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# cluster-size-read-fraction-statistics

## Summary

Computes how reads are distributed across UMI-read clusters of different sizes (e.g., fraction of reads in size-one clusters versus shifted clusters) pooled per dataset, quantifying the extent of the UMI mapping-shift artefact for RNA-Seq UMI-counting analysis.

## When to use

Use when FASTQ-to-SAM UMI pipelines have produced per-position UMI cluster data for one or more UMI-based RNA-Seq datasets and you need to quantify what proportion of reads fall into size-1 clusters versus mapping-shifted clusters.

## When NOT to use

- Input data contains no UMIs — the article states 'In the absence of UMI this artefact is undetectable', so cluster-size statistics are undefined
- The goal is paired-end or splice-aware transcript-level quantification — the pipeline is designed for single-ended sequencing with STAR specifying no introns
- You need the true biological distribution of mapping-shift sizes — the article notes the 3 bp collapse threshold is arbitrary and the true distribution is unknown

## Inputs

- Demultiplexed single-end FASTQ files per dataset (e.g., from SRA accessions for run_171108, run_170420, SCRB, La Manno, Yanai1, Yanai2)
- Sorted SAM files from STAR alignment (no introns)
- position_umi_counts.bed (unique UMI counts per genomic position from umi_clusterizer.pl)
- umi_positions.txt (UMI-to-genomic-position mapping from umi_analyzer_uthash)

## Outputs

- Per-dataset table of read fractions by UMI-read cluster size (e.g., Table 1-equivalent: ~19% size-1 for run_171108, ~75% for La Manno)
- Pooled UMI-read cluster mapping-shift density distributions per dataset (Figs. 2–4 equivalents)
- all_sam_umis.txt and collapsed_sam_umis.txt logging files from umi_counterizer.pl

## How to apply

For each dataset (e.g., run_171108, run_170420, SCRB, La Manno, Yanai1, Yanai2), quality-filter the FASTQ with fastq_qual_filter (min_ok ~30), clip the 5' UMI with fastq_umi_clipper (g_length set per protocol to remove trailing Gs from SMRTER template-switch libraries), align with STAR specifying no introns to get a sorted SAM, then run umi_counterizer.pl with a max edit distance of 3 bp so UMIs within 3 bp at the same mapping position collapse into one. Follow with umi_analyzer_uthash to map UMIs to genomic positions (umi_positions.txt) and umi_clusterizer.pl to count unique UMIs per position (position_umi_counts.bed). Pool UMI-read clusters across each dataset and compute, per cluster size, the total number of reads contained in clusters of that size — the article treats reads separated by ≤3 bp sharing a UMI as derived from the same starting mRNA, so the size-1 fraction approximates unshifted reads. Rationale: the 3 bp threshold is a configurable parameter because the true distribution of mapping shifts is unknown.

## Related tools

- **fastq_qual_filter** (C program that quality-filters FASTQ reads on a substring (e.g., UMI at min_ok 30, offset 0, length 10) before UMI clipping) — https://github.com/ncgr/UMI-analysis
- **fastq_umi_clipper** (C program that moves the 5' UMI into the FASTQ header and deletes protocol-derived trailing Gs (g_length) before alignment) — https://github.com/ncgr/UMI-analysis
- **STAR** (Aligns clipped reads to the genomic reference specifying no introns, producing the sorted SAM needed for UMI collapsing) — https://github.com/alexdobin/STAR
- **umi_counterizer.pl** (Perl script collapsing duplicate UMIs at the same mapping position with a max_edit_distance of 3 bp and counting UMIs per position) — https://github.com/ncgr/UMI-analysis
- **umi_analyzer_uthash** (C program using uthash to find all genomic positions where each UMI maps (umi_positions.txt), the key input for cluster-size pooling) — https://github.com/ncgr/UMI-analysis
- **umi_clusterizer.pl** (Perl script counting unique UMIs per genomic position into position_umi_counts.bed, the basis of the cluster-size read-fraction statistics) — https://github.com/ncgr/UMI-analysis
- **bedtools** (Optionally intersects position_umi_counts.bed with GFF gene annotations to confirm gene-level behaviour of the clusters)

## Examples

```
umi_counterizer.pl sorted_samfile 3 > position_umi_counts.txt && bedtools intersect -a position_umi_counts.bed -b file.gff -wa -wb | pcregrep '\tgene\t' > umi_bed_intersection_gff.txt
```

## Evaluation signals

- Reproduced size-1 cluster read fraction for run_171108 matches the reported ~19% (majority of reads in shifted clusters)
- Reproduced size-1 cluster read fraction for the La Manno dataset matches the reported ~75%
- Mapping-shift density distributions peak at zero shift with a bell-shaped decay around the modal position
- collapsed_sam_umis.txt contains a non-redundant list of UMIs and all_sam_umis.txt contains every UMI observed in the SAM file
- Intersecting position_umi_counts.bed with GFF gene annotations yields coherent gene-level UMI counts via bedtools intersect and gene_umi_counter.pl

## Limitations

- The 3 bp separation threshold is a configurable parameter chosen arbitrarily; reads derived from the same template could have larger mapping shifts, the true distribution being unknown
- Positions of UMIs mapping to multiple genomic locations complicate attribution; scripts like umi_analyzer_uthash have limited error checking and strict input-structure assumptions (space-delimited, fixed UMI length)
- The scripts warn 'not much arg checking is done so unexpected behavior may be encountered' — e.g., substrings longer than read length, negative values, or silent overwriting of output files
- Data sets may be too small to observe meaningful trends in entropy metrics and repeat content related to the artefact

## Evidence

- [discussion] we considered reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs: "we considered reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs"
- [discussion] reads derived from the same template could have larger mapping shifts, the true distribution being unknown. Accordingly, we make this separation a configurable parameter: "reads derived from the same template could have larger mapping shifts, the true distribution being unknown. Accordingly, we make this separation a configurable parameter"
- [readme] Run the Perl script umi_counterizer.pl on your sorted SAM file. This requires the packages Text::Levenshtein::XS and Try::Tiny: "Run the Perl script umi\_counterizer.pl on your sorted SAM file. This requires the packages [Text::Levenshtein::XS]"
- [abstract] In the absence of UMI this artefact is undetectable.: "In the absence of UMI this artefact is undetectable."
- [methods] Warning: not much arg checking is done so unexpected behavior may be encountered.: "Warning: not much arg checking is done so unexpected behavior may be encountered."
