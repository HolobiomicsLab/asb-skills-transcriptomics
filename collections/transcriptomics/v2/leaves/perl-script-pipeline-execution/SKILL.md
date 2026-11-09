---
name: perl-script-pipeline-execution
description: Use when a sorted SAM file or umi_positions.txt from a UMI-tagged RNA-Seq
  run must be deduplicated, with reads sharing a UMI mapping within 3 bp grouped into
  UMI-read clusters and counted per position or per gene.
when_to_use_negative:
- Input has no UMI information (plain FASTQ/FASTA or untagged RNA-Seq), since the
  artefact being corrected is undetectable without UMI and the scripts depend on UMI
  strings in headers or mappings.
- You only need duplicate collapsing at identical genomic positions and already have
  that done by another deduplicator; this skill's core value is grouping same-UMI
  reads across closely spaced (≤3 bp) positions into UMI-read clusters.
- The input to umi_counterizer.pl is not a sorted SAM file — the scripts make strict
  assumptions about input structure and do little argument or error checking.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Perl
  role: Language in which the pipeline scripts (umi_counterizer.pl, umi_clusterizer.pl,
    gene_umi_counter.pl) are implemented and executed
  repo: github.com/ncgr/UMI-analysis
- name: Text::Levenshtein::XS
  role: CPAN package required by umi_counterizer.pl to compute UMI edit distances
    when collapsing duplicates at the same position
- name: Try::Tiny
  role: CPAN package required alongside Text::Levenshtein::XS by umi_counterizer.pl
- name: bedtools
  role: Intersects position_umi_counts.bed with a GFF annotation file before per-gene
    counting
- name: STAR
  role: Produces the sorted SAM file of aligned, UMI-clipped reads that umi_counterizer.pl
    consumes (run specifying no introns)
  repo: github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:f801acfd3f7cdf52816b6afe5da428f21ae4d407bc9cfae3992859ed7bc8ae90
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Perl
  - STAR
  - Text::Levenshtein::XS
  - Try::Tiny
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# perl-script-pipeline-execution

## Summary

Execute the Perl scripts of the ncgr/UMI-analysis pipeline (umi_counterizer.pl, umi_clusterizer.pl, gene_umi_counter.pl) to collapse UMI-read clusters and produce per-position and per-gene UMI counts, addressing the read mapping shift artefact in UMI-based RNA-Seq quantification.

## When to use

Use when a sorted SAM file or umi_positions.txt from a UMI-tagged RNA-Seq run must be deduplicated, with reads sharing a UMI mapping within 3 bp grouped into UMI-read clusters and counted per position or per gene.

## When NOT to use

- Input has no UMI information (plain FASTQ/FASTA or untagged RNA-Seq), since the artefact being corrected is undetectable without UMI and the scripts depend on UMI strings in headers or mappings.
- You only need duplicate collapsing at identical genomic positions and already have that done by another deduplicator; this skill's core value is grouping same-UMI reads across closely spaced (≤3 bp) positions into UMI-read clusters.
- The input to umi_counterizer.pl is not a sorted SAM file — the scripts make strict assumptions about input structure and do little argument or error checking.

## Inputs

- sorted SAM file with UMI embedded in read headers (from fastq_umi_clipper and STAR alignment)
- umi_positions.txt: UMI-to-genomic-position mapping from umi_analyzer_uthash (UMI followed by space-delimited positions)
- position_umi_counts.bed (chromosome, start, end, unique UMI count)
- GFF annotation file for intersecting UMI positions with genes

## Outputs

- STDOUT lines of 'position<TAB>number of UMI<TAB>UMIs (space delimited)', e.g. 'Chr01:219196 6 TGATGGAGGG ...'
- all_sam_umis.txt and collapsed_sam_umis.txt logs (sorted lists of observed and non-redundant UMIs)
- position_umi_counts.bed: BED lines such as 'Chr01\t10070498\t10070498\t6' giving unique UMI count per position
- umi_bed_intersection_gff.txt (bedtools intersect output filtered with pcregrep for '\tgene\t') and per-gene UMI/read counts

## How to apply

After alignment (e.g. STAR with no introns) and UMI clipping, run umi_counterizer.pl on the sorted SAM file with a max_edit_distance argument so UMIs exceeding that Levenshtein distance at the same genomic position are kept distinct rather than collapsed; it uses Text::Levenshtein::XS and Try::Tiny and prints position, number of UMI, and the UMIs to STDOUT, while logging all_sam_umis.txt and collapsed_sam_umis.txt. Feed that output to umi_analyzer_uthash (C/uthash) to obtain umi_positions.txt, where each line lists a UMI followed by its space-delimited mapping positions. Then run umi_clusterizer.pl on umi_positions.txt: mapping positions sharing a UMI and separated by three or fewer base pairs (a configurable threshold, 3 bp in the paper) are grouped into UMI-read clusters and each cluster is collapsed into a single UMI observation, since reads with different mapping coordinates can derive from the same starting molecule. Finally, write unique-UMI counts per position in BED format (chromosome, start, end, count) to position_umi_counts.bed, which can be intersected with a GFF via bedtools and summarized with gene_umi_counter.pl to obtain per-gene counts.

## Related tools

- **Perl** (Language in which the pipeline scripts (umi_counterizer.pl, umi_clusterizer.pl, gene_umi_counter.pl) are implemented and executed) — github.com/ncgr/UMI-analysis
- **Text::Levenshtein::XS** (CPAN package required by umi_counterizer.pl to compute UMI edit distances when collapsing duplicates at the same position)
- **Try::Tiny** (CPAN package required alongside Text::Levenshtein::XS by umi_counterizer.pl)
- **bedtools** (Intersects position_umi_counts.bed with a GFF annotation file before per-gene counting)
- **STAR** (Produces the sorted SAM file of aligned, UMI-clipped reads that umi_counterizer.pl consumes (run specifying no introns)) — github.com/alexdobin/STAR

## Examples

```
perl umi_clusterizer.pl umi_positions.txt  # writes position_umi_counts.bed (chromosome\tstart\tend\tunique UMI count)
```

## Evaluation signals

- position_umi_counts.bed exists and every line has exactly four tab-delimited fields with a non-negative integer in the fourth column (e.g. 'Chr01\t10070498\t10070498\t6').
- umi_counterizer.pl STDOUT lines match the format 'position number_of_UMI UMI1 UMI2 ...' with space-delimited UMIs of the expected length (e.g. 10 nt in the examples).
- Logs all_sam_umis.txt and collapsed_sam_umis.txt are produced, sorted, with collapsed_sam_umis.txt being the non-redundant subset of all_sam_umis.txt.
- Same-UMI mapping positions separated by more than 3 bp remain distinct observations after clustering, while those within 3 bp collapse into a single UMI observation.
- Counts per gene from gene_umi_counter.pl should not exceed collapsed UMI counts, since uncollapsed data is the 'case' that over-estimates transcript abundance.

## Limitations

- The 3 bp cluster-separation threshold is arbitrary and configurable because the true distribution of mapping shift sizes is unknown; reads derived from the same template could have larger shifts.
- The scripts perform little argument or error checking ('not much arg checking is done so unexpected behavior may be encountered'); output files are overwritten without warning and inputs must strictly match expected formats.
- Not finished software: users must be able to read/edit C and Perl, install libraries (Better String Library, uthash, Text::Levenshtein::XS, Try::Tiny), edit makefiles, and set LD_LIBRARY_PATH; the pipeline assumes single-ended sequencing only.
- Processing large FASTQ files in Perl is slow, which is why the upstream FASTQ steps are in C rather than Perl.

## Evidence

- [readme] Run the Perl script umi_counterizer.pl on your sorted SAM file. This requires the packages Text::Levenshtein::XS and Try::Tiny.: [evidence span withheld — anchor status: not_found]
- [readme] At the same mapping position on the genome, UMI greater than max_edit_distance from each other will be considered different UMI and not collapsed into one.: [evidence span withheld — anchor status: external_doc]
- [discussion] we considered reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs: "we considered reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs"
- [methods] We use STAR specifying no introns. To get to the next step you need a sorted SAM file.: "We use STAR specifying no introns. To get to the next step you need a sorted SAM file."
- [methods] Execute the perl script gene_umi_counter.pl on the above output:: [evidence span withheld — anchor status: external_doc]
