---
name: bed-format-count-output-generation
description: Use when you have a UMI-to-genomic-position mapping (umi_positions.txt)
  and need position-level UMI counts in BED format for downstream annotation intersection.
when_to_use_negative:
- Input is not a UMI-to-position mapping (e.g. you only have a sorted SAM file) —
  first run umi_counterizer.pl and umi_analyzer_uthash to produce umi_positions.txt
- You need read counts per gene rather than per-position UMI counts — later steps
  (bedtools intersect plus gene_umi_counter.pl) handle that
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: Perl
  role: Language of umi_clusterizer.pl, the script that clusters UMI mapping positions
    and writes position_umi_counts.bed
- name: bedtools
  role: 'Downstream consumer: intersects position_umi_counts.bed with a GFF file to
    find genes matching UMI positions'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:e6b3f67aae826e0689e225e060897b82379cf979f06e8268b916b4bfbec385e6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Perl
  - bedtools
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# bed-format-count-output-generation

## Summary

Generates a per-genomic-position BED file of unique UMI counts (chromosome, start, end, count) using umi_clusterizer.pl, so that UMI counts collapsed from UMI-read clusters can be intersected with GFF annotations.

## When to use

Use when you have a UMI-to-genomic-position mapping (umi_positions.txt) and need position-level UMI counts in BED format for downstream annotation intersection.

## When NOT to use

- Input is not a UMI-to-position mapping (e.g. you only have a sorted SAM file) — first run umi_counterizer.pl and umi_analyzer_uthash to produce umi_positions.txt
- You need read counts per gene rather than per-position UMI counts — later steps (bedtools intersect plus gene_umi_counter.pl) handle that

## Inputs

- umi_positions.txt: UMI-to-genomic-position mapping from umi_analyzer_uthash, one UMI per line followed by space-delimited mapping positions (e.g. 'TGTAAATTGG Chr01:76001 Chr01:76002')
- cluster-separation threshold parameter (3 bp used in the paper, configurable)

## Outputs

- position_umi_counts.bed: BED-format file with four tab-delimited fields per line (chromosome, start coordinate, end coordinate, unique UMI count), e.g. 'Chr01	10070498	10070498	6'

## How to apply

Run the Perl script umi_clusterizer.pl on umi_positions.txt, the UMI-to-genomic-position mapping produced by umi_analyzer_uthash where each line lists a UMI followed by its space-delimited mapping positions. The script groups mapping positions sharing a UMI that are separated by three or fewer base pairs (the configurable cluster-separation threshold used in the paper) into UMI-read clusters, collapsing each cluster into a single UMI observation. It then counts unique UMIs found in reads mapping to each genomic position and writes the counts in BED format (chromosome, start coordinate, end coordinate, unique UMI count) to position_umi_counts.bed. The BED output is designed to feed into bedtools intersect against a GFF to assign UMI counts to genes.

## Related tools

- **Perl** (Language of umi_clusterizer.pl, the script that clusters UMI mapping positions and writes position_umi_counts.bed)
- **bedtools** (Downstream consumer: intersects position_umi_counts.bed with a GFF file to find genes matching UMI positions)

## Examples

```
perl umi_clusterizer.pl umi_positions.txt > position_umi_counts.bed
```

## Evaluation signals

- position_umi_counts.bed exists and is non-empty
- Every line contains exactly four tab-delimited fields
- The fourth column is a non-negative integer unique UMI count
- Coordinates are well-formed BED intervals (start ≤ end; in the exemplified output start equals end, e.g. 'Chr01	10070498	10070498	6')
- Counted positions are consistent with positions present in the input umi_positions.txt

## Limitations

- The 3 bp cluster-separation threshold is an arbitrary configurable parameter; the true distribution of mapping shift sizes is unknown, so reads derived from the same template could have larger mapping shifts
- The README warns that the pipeline programs do little argument checking, so unexpected behavior may be encountered
- Only single-ended sequencing was exercised in the authors' use case

## Evidence

- [discussion] we considered reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs: "we considered reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs"
- [methods] This is done by taking the positions of the UMI in the previous step and finding the corresponding annotations in a GFF file.: "This is done by taking the positions of the UMI in the previous step and finding the corresponding annotations in a GFF file. Run bedtools thus"
- [discussion] reads derived from the same template could have larger mapping shifts, the true distribution being unknown. Accordingly, we make this separation a configurable parameter: "reads derived from the same template could have larger mapping shifts, the true distribution being unknown. Accordingly, we make this separation a configurable parameter"
- [readme] Output is each UMI and where it maps in the genome. This file is usually called umi_positions.txt.: "Output is each UMI and where it maps in the genome. This file is usually called umi_positions.txt."
- [readme] Warning: not much arg checking is done so unexpected behavior may be encountered.: "Warning: not much arg checking is done so unexpected behavior may be encountered."
