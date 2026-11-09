---
name: exon-intron-structure-annotation
description: Use when PHAS loci predicted from sRNA-seq data must be annotated by
  exon/intron structure against a genome gff3, especially when comparing transcriptome-based
  vs genome-based phasiRNA prediction results.
when_to_use_negative:
- No genome gff3 annotation is available for the organism, since exon/intron classification
  relies on the gff3 and Bedtools overlap
- The goal is de novo phasiRNA prediction itself rather than structural annotation/comparison
  of already-predicted PHAS loci
- The loci of interest come from a tool whose coordinates are not compatible with
  Bedtools interval overlap against the chosen gff3 release
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: PhasiHunter
  role: Runs the preprocess, phase, and integration submodules whose gff3-aware integration
    step enables gdna-based PHAS loci and supports structural classification
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bedtools
  role: Interval overlap against gff3 exon/intron features to classify PHAS loci as
    intron+exon-spanning, intronic, or exonic
  repo: https://github.com/HuangLab-CBI/PhasiHunter
- name: Bowtie
  role: Maps filtered sRNA reads to both transcriptome and genome indexes to build
    the .map files feeding phase prediction
  repo: https://github.com/HuangLab-CBI/PhasiHunter
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad676
    title: 'PhasiHunter: a robust phased siRNA regulatory cascade mining tool based
      on multiple reference sequences'
  source_content_hash: sha256:60ada9eacfa54c743a71733b0c469987b25c6b7aa552b9cdbaf87711b930ddee
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# exon-intron-structure-annotation

## Summary

Classifies predicted PHAS/phasiRNA loci by their exon/intron structure using genome gff3 annotation and Bedtools interval overlap, enabling comparison of transcriptome-based vs genome-based phasiRNA predictions (e.g., mono-exon, multi-exon, intron+exon-spanning, intronic loci).

## When to use

Use when PHAS loci predicted from sRNA-seq data must be annotated by exon/intron structure against a genome gff3, especially when comparing transcriptome-based vs genome-based phasiRNA prediction results.

## When NOT to use

- No genome gff3 annotation is available for the organism, since exon/intron classification relies on the gff3 and Bedtools overlap
- The goal is de novo phasiRNA prediction itself rather than structural annotation/comparison of already-predicted PHAS loci
- The loci of interest come from a tool whose coordinates are not compatible with Bedtools interval overlap against the chosen gff3 release

## Inputs

- phase_o.txt and phase_a.txt outputs from PhasiHunter phase module (transcriptome-based and genome-based)
- Genome gff3 annotation file (e.g., oryza_sativa_gdna.gff3)
- Processed sRNA map files (*_processed_cdna.map, *_processed_gdna.map) from PhasiHunter preprocess
- sRNA fasta file (e.g., *_trimmed_format_filter.fa)
- Reference transcriptome and genome fasta files

## Outputs

- integration_o.txt, integration_a.txt, integration_s.txt, integration_p.txt from PhasiHunter integration
- Set of transcriptome-unique and genome-unique PHAS genes
- Per-locus exon/intron structure classification (mono-exon, multi-exon, intron+exon-spanning, intronic)
- Comparison summary table (e.g., phas_loci_comparison_summary.tsv) mirroring Supplementary Fig. S3

## How to apply

Run PhasiHunter preprocess on each sRNA-seq library (length filter 19-25 nt, CPM cutoff 1, normalization base 1000000, max hits 10) mapping reads to both the reference transcriptome (Bowtie cDNA index) and reference genome (gdna index), producing *_processed_cdna.map and *_processed_gdna.map files. Run the phase module separately per reference with both prediction methods (-me b), phase length 21, phase number cutoff 4, cluster island 5, p-value cutoff 0.001, phase score cutoff 15, and phase ratio cutoff 0.4, yielding phase_a.txt and phase_o.txt for each reference. Integrate results with the PhasiHunter integration module using the genome gff3 annotation (-an), enabling gdna-based PHAS loci (-g y) and discarding only-P-method results (-dp y). Then compute the set difference of PHAS genes between the two references to isolate reference-unique loci, and classify them by exon/intron span using the gff3 with Bedtools interval overlap: genome-unique loci are labeled as spanning intron+exon, exonic, or intronic, while transcriptome-unique loci are labeled mono-exon vs multi-exon. In the maize 32-library test case this yielded 30 transcriptome-unique PHAS genes (60% multi-exon, 40% mono-exon) and 95 genome-unique PHAS genes (69.5% spanning intron and exon, 1% intronic).

## Related tools

- **PhasiHunter** (Runs the preprocess, phase, and integration submodules whose gff3-aware integration step enables gdna-based PHAS loci and supports structural classification) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bedtools** (Interval overlap against gff3 exon/intron features to classify PHAS loci as intron+exon-spanning, intronic, or exonic) — https://github.com/HuangLab-CBI/PhasiHunter
- **Bowtie** (Maps filtered sRNA reads to both transcriptome and genome indexes to build the .map files feeding phase prediction) — https://github.com/HuangLab-CBI/PhasiHunter

## Examples

```
phasiHunter integration -io phase_o.txt -ia phase_a.txt -an oryza_sativa_gdna.gff3 -g y -o integration_o.txt -a integration_a.txt -s integration_s.txt -po integration_p.txt -pn 4 -pl 21 -pv 0.001 -il 5
```

## Evaluation signals

- Unique-loci counts match reported values: 30 transcriptome-unique and 95 genome-unique PHAS genes on the 32-library maize sRNA-seq test dataset
- Transcriptome-unique loci classification matches 60% multi-exon / 40% mono-exon
- Genome-unique loci classification matches 69.5% intron+exon-spanning and 1% intronic
- integration_o.txt, integration_a.txt, integration_s.txt, and integration_p.txt are all produced by the integration module
- Summary table (phas_loci_comparison_summary.tsv) contains the required classification fields (exon/intron span categories per locus set)

## Limitations

- Only-P-method results are discarded by default in integration (-dp default=y), so hypergeometric-only evidence drives the reported locus sets
- The published version numbering is limited (update log only reaches 1.0.1, dated 25/02/2024), and no discussion-section limitations were extractable
- Percentages depend on gff3 annotation completeness; loci without reliable exon/intron models cannot be classified cleanly

## Evidence

- [abstract] PhasiHunter can identify phasiRNAs and their regulatory networks based on multiple reference sequences and the predicted results can be automatically integrated: "PhasiHunter can identify phasiRNAs and their regulatory networks based on multiple reference sequences and the predicted results can be automatically integrated"
- [readme] Phase prediction methods: "-me: str   --  phasiRNA prediction method, h(hypergeometric test) | p(phase score) | b (both), default=b"
- [readme] Preprocess filter chain: "raw(mode): trim adaptor --> normalization --> length and abundance filter --> mapping"
- [readme] Integration command with gff3 annotation: "phasiHunter integration -io /home/user/test_osa/phase_o.txt -ia /home/user/test_osa/phase_a.txt -an /home/user/test_osa/oryza_sativa_gdna.gff3 -g y"
- [methods] Discard only-P-method results: "-dp: str   -- y | n, discard only P method result, default=y"
