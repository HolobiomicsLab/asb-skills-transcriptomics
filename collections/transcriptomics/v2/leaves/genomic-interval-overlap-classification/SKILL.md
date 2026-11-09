---
name: genomic-interval-overlap-classification
description: Use when phasiRNA/PHAS loci predicted against both a transcriptome and
  a genome reference must be de-duplicated into reference-unique sets and each unique
  locus classified by exon/intron structure from a gff3 annotation (e.g. mirroring
  Supplementary Fig. S3).
when_to_use_negative:
- Input is a single-reference-only phasiRNA prediction with no paired transcriptome/genome
  result sets to take set differences over
- No genome gff3 annotation is available, so exon/intron span classification cannot
  be performed
- The goal is phasiRNA initiator/target prediction (degradome, T-plot, eTM search)
  rather than PHAS locus structural classification
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3308
tools:
- name: PhasiHunter
  role: Runs the phase and integration submodules that generate per-reference PHAS
    loci sets and merge them against the genome gff3 before classification
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bedtools
  role: Interval overlap of genome-unique PHAS loci with gff3 exon/intron features
    to compute span percentages
- name: Perl5
  role: Scripting the set-difference of PHAS gene sets and writing the comparison
    summary table
  repo: https://www.perl.org
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:cb176504cbd2f728214eed0f6687249f1801ac1f81f5380211cb3845c387ca60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuangLab-CBI/PhasiHunter
  tools:
  - Bedtools
  - Bowtie
  - Perl5
  - PhasiHunter
  - Seqkit
  - Snakemake
  - groHMM
  - rtracklayer
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# genomic-interval-overlap-classification

## Summary

Classify reference-unique PHAS loci by their exon/intron span by intersecting phasiRNA-predicted gene sets with a genome gff3 annotation using Bedtools, then compare transcriptome- vs genome-based prediction yields. In the PhasiHunter maize test case this resolves 30 transcriptome-unique vs 95 genome-unique PHAS genes with distinct structural compositions.

## When to use

Use when phasiRNA/PHAS loci predicted against both a transcriptome and a genome reference must be de-duplicated into reference-unique sets and each unique locus classified by exon/intron structure from a gff3 annotation (e.g. mirroring Supplementary Fig. S3).

## When NOT to use

- Input is a single-reference-only phasiRNA prediction with no paired transcriptome/genome result sets to take set differences over
- No genome gff3 annotation is available, so exon/intron span classification cannot be performed
- The goal is phasiRNA initiator/target prediction (degradome, T-plot, eTM search) rather than PHAS locus structural classification

## Inputs

- phase module outputs phase_o.txt (phasiRNA clusters) and phase_a.txt (allsiRNA clusters) for both cDNA-based and gdna-based runs
- processed map files (*_processed_cdna.map, *_processed_gdna.map) from PhasiHunter preprocess
- reference transcriptome and genome fasta files (e.g. oryza_sativa_cdna.fa, oryza_sativa_gdna.fa)
- genome gff3 annotation file (e.g. oryza_sativa_gdna.gff3)

## Outputs

- integration_o.txt, integration_a.txt, integration_s.txt, integration_p.txt (PhasiHunter integration module)
- reference-unique PHAS gene sets (transcriptome-unique and genome-unique gene lists)
- exon/intron classification table phas_loci_comparison_summary.tsv (counts + % multi-exon/mono-exon, % intron+exon-spanning, % intronic)

## How to apply

First run the PhasiHunter phase module separately on cDNA-based (preprocess -m r) and gdna-based (preprocess -m m) .map files with both prediction methods (-me b), phase length 21 (-pl 21), phase number cutoff 4 (-pn 4), cluster island 5 (-il 5), p-value cutoff 0.001 (-pv 0.001), phase score cutoff 15 (-ps 15), and phase ratio cutoff 0.4 (-pr 0.4), producing phase_a.txt and phase_o.txt per reference. Then merge both result sets with the PhasiHunter integration module against the genome gff3 (oryza_sativa_gdna.gff3, -g y, -pn 4, -pl 21, -pv 0.001, -il 5), keeping -dp y to discard only-P-method results. Compute the set difference of PHAS genes between the transcriptome-based and genome-based results to obtain each reference's unique gene set. Classify genome-unique loci by exon/intron span using Bedtools interval overlap with the gff3 features (intron+exon-spanning vs intronic), and classify transcriptome-unique loci as mono- vs multi-exon transcript status. Write counts and classifications to phas_loci_comparison_summary.tsv. Rationale: the integration step's gff3-backed annotation enables structural classification, and only hypergeometric-supported (non-only-P) loci are retained to keep predictions robust.

## Related tools

- **PhasiHunter** (Runs the phase and integration submodules that generate per-reference PHAS loci sets and merge them against the genome gff3 before classification) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bedtools** (Interval overlap of genome-unique PHAS loci with gff3 exon/intron features to compute span percentages)
- **Perl5** (Scripting the set-difference of PHAS gene sets and writing the comparison summary table) — https://www.perl.org

## Examples

```
phasiHunter integration -io phase_o.txt -ia phase_a.txt -an oryza_sativa_gdna.gff3 -g y -o integration_o.txt -a integration_a.txt -s integration_s.txt -po integration_p.txt -pn 4 -pl 21 -pv 0.001 -il 5
```

## Evaluation signals

- Unique PHAS gene counts match reported values: 30 transcriptome-unique and 95 genome-unique loci
- Genome-unique loci classification reproduces 69.5% intron+exon-spanning and 1% intronic; transcriptome-unique loci reproduce 60% multi-exon / 40% mono-exon
- phas_loci_comparison_summary.tsv contains the required classification fields (unique-loci counts and exon/intron categories)
- Integration parameters (-pn 4, -pl 21, -pv 0.001, -il 5, -dp y) match the phase module settings so only hypergeometric-supported loci are merged
- Each unique-locus set is disjoint from the other reference's PHAS gene set (verified set difference)

## Limitations

- Only-P-method results are discarded during integration (-dp default y), so loci supported solely by phase score are excluded from classification
- Classification quality depends on the completeness/correctness of the gff3 annotation; loci without gff3 overlap cannot be span-classified
- PhasiHunter's changelog lists only version 1.0.1 (25/02/2024) and no discussion-section limitations or reproducibility details were available

## Evidence

- [readme] Phase parameters govern which loci enter the comparison: "-me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b"
- [readme] Integration against genome gff3 with matching cutoffs enables classification: "phasiHunter integration -io /home/user/test_osa/phase_o.txt -ia /home/user/test_osa/phase_a.txt -an /home/user/test_osa/oryza_sativa_gdna.gff3 -g y"
- [methods] Only-P results are discarded, restricting the classified locus set: "-dp: str   -- y | n, discard only P method result, default=y"
- [methods] Integration is the third workflow step producing the merged locus sets: "3. PhasiRNA and PHAS loci result integration"
- [methods] Bedtools is the named tool for interval operations in the pipeline: "Bedtools (Quinlan and Hall, 2010. Bioinformatics)"
