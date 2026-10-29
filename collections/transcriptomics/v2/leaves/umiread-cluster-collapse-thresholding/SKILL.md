---
name: umiread-cluster-collapse-thresholding
description: Use when UMI-to-genomic-position mappings (e.g. umi_positions.txt from
  umi_analyzer_uthash) must be grouped and collapsed so reads sharing a UMI that map
  within 3 bp count once instead of as multiple molecules.
when_to_use_negative:
- Input is a sorted SAM file needing same-position duplicate elimination — use umi_counterizer.pl
  with a max_edit_distance instead, since cluster-collapse operates on already-extracted
  UMI-position mappings
- Input is the collapsed BED already intersected with GFF annotations and only per-gene
  UMI/read totals are needed — use gene_umi_counter.pl rather than re-clustering
- Reads without UMIs (UMI-free RNA-Seq), where the mapping-shift artefact is undetectable
  and collapse cannot be applied
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: Perl
  role: Runs umi_clusterizer.pl, which groups UMI-sharing mapping positions within
    the 3 bp threshold into UMI-read clusters and writes per-position UMI counts in
    BED format
  repo: https://github.com/ncgr/UMI-analysis
- name: uthash
  role: Underlies the C program umi_analyzer_uthash that produces umi_positions.txt,
    the required UMI-to-genomic-position input for cluster collapse
  repo: https://github.com/troydhanson/uthash
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:0f69ec37e4cda11639f5491b0ff1c6e608ad33c6186f0bc300a275c391a4bd3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncgr/UMI-analysis
  tools:
  - Perl
  - uthash
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# umiread-cluster-collapse-thresholding

## Summary

Groups reads that share a UMI and map to genomic coordinates within 3 bp of one another into UMI-read clusters, then collapses each cluster into a single UMI observation to prevent over-counting caused by the mapping-shift artefact.

## When to use

Use when UMI-to-genomic-position mappings (e.g. umi_positions.txt from umi_analyzer_uthash) must be grouped and collapsed so reads sharing a UMI that map within 3 bp count once instead of as multiple molecules.

## When NOT to use

- Input is a sorted SAM file needing same-position duplicate elimination — use umi_counterizer.pl with a max_edit_distance instead, since cluster-collapse operates on already-extracted UMI-position mappings
- Input is the collapsed BED already intersected with GFF annotations and only per-gene UMI/read totals are needed — use gene_umi_counter.pl rather than re-clustering
- Reads without UMIs (UMI-free RNA-Seq), where the mapping-shift artefact is undetectable and collapse cannot be applied

## Inputs

- umi_positions.txt — UMI-to-genomic-position mapping (one UMI per line, followed by space-delimited positions, e.g. 'TGTAAATTGG Chr01:76001 Chr01:76002'), output of umi_analyzer_uthash
- Cluster-separation threshold in bp (paper default: 3)

## Outputs

- position_umi_counts.bed — BED-format per-position unique UMI counts (chromosome, start coordinate, end coordinate, unique UMI count)
- Set of collapsed UMI-read clusters usable as single observations for downstream counting

## How to apply

Load umi_positions.txt, the UMI-to-genomic-position mapping where each line lists a UMI followed by its space-delimited mapping positions, produced by the uthash-based C program umi_analyzer_uthash. Run the Perl script umi_clusterizer.pl on this file; it groups mapping positions that share a UMI and are separated by three or fewer base pairs — the separation threshold used in the paper — into UMI-read clusters, collapsing each cluster into one UMI observation. The 3 bp value is a configurable parameter because the true distribution of mapping shifts is unknown and reads from the same template could shift farther. After clustering, count unique UMIs at each genomic position and write per-position counts in BED format (chromosome, start, end, unique UMI count) to position_umi_counts.bed. Rationale: treating each distinct mapping coordinate as an independent starting molecule over-estimates transcript abundance for reads exhibiting the mapping-shift artefact.

## Related tools

- **Perl** (Runs umi_clusterizer.pl, which groups UMI-sharing mapping positions within the 3 bp threshold into UMI-read clusters and writes per-position UMI counts in BED format) — https://github.com/ncgr/UMI-analysis
- **uthash** (Underlies the C program umi_analyzer_uthash that produces umi_positions.txt, the required UMI-to-genomic-position input for cluster collapse) — https://github.com/troydhanson/uthash

## Examples

```
perl umi_clusterizer.pl umi_positions.txt > position_umi_counts.bed
```

## Evaluation signals

- position_umi_counts.bed exists and every line contains exactly four tab-delimited fields
- Fourth field of each BED line is a non-negative integer unique-UMI count
- Output matches the exemplified format, e.g. 'Chr01\t10070498\t10070498\t6'
- Each output position corresponds to one collapsed UMI-read cluster (no line per raw mapping coordinate within a ≤3 bp window of the same UMI)
- Total unique-UMI counts after collapse are no greater than counts before collapse at the same locus

## Limitations

- The 3 bp separation threshold is an arbitrary configurable parameter; reads from the same template could have larger mapping shifts since the true shift distribution is unknown
- The input umi_positions.txt has a rigid structure ('no flexibility here, and not much error checking'); assumptions about UMI length, position-string length, and line length must be configured before compiling umi_analyzer_uthash
- Cluster collapse assumes single-ended sequencing data, as in the authors' use case

## Evidence

- [discussion] Reads sharing a UMI with mapping locations separated by ≤3 bp considered derived from the same starting mRNA: "we considered reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs"
- [discussion] 3 bp collapse threshold is a configurable parameter: "reads derived from the same template could have larger mapping shifts, the true distribution being unknown. Accordingly, we make this separation a configurable parameter"
- [discussion] Pipeline groups closely-spaced read clusters as single observations: "our analysis pipeline, which groups clusters of closely-spaced reads together as single observations"
- [readme] umi_positions.txt produced by umi_analyzer_uthash is the UMI-to-position mapping: "Output is each UMI and where it maps in the genome. This file is usually called umi_positions.txt."
- [abstract] Reads sharing a UMI map to closely spaced locations: "reads sharing a UMI, and therefore presumed to be derived from the same mRNA molecule, frequently map to different, but closely spaced locations."
