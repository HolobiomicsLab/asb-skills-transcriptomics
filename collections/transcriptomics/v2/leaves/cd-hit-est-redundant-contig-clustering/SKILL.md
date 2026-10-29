---
name: cd-hit-est-redundant-contig-clustering
description: Use when a de novo transcriptome assembly (e.g. Trinity output in FASTA)
  contains redundant, highly identical contigs that must be deduplicated as part of
  the Bellerophon filtration sequence (TPM -> CD-HIT-EST -> TransRate-C).
when_to_use_negative:
- When a custom filtering order is required — the README states 'Custom order of filtering
  steps is disabled in this version', so CD-HIT-EST cannot be repositioned within
  the current pipeline.
- When the goal is removing low-expression contigs — that is the TPM filter's role
  (cut-off 1.0); CD-HIT-EST targets sequence redundancy, not expression level.
- When ORF-length trimming is acceptable instead — the authors set the default order
  to TPM then CD-HIT-EST 'as the ORF-length filtering appeared to be removing too
  much transcripts'.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: CD-HIT-EST
  role: Clusters and removes highly identical/redundant contigs at a default 95% identity
    cut-off as the second filtration step of Bellerophon
- name: Bellerophon
  role: Pipeline that orchestrates the CD-HIT-EST step after TPM filtering and before
    TransRate-C, and generates the results report
  repo: github.com/JesseKerkvliet/Bellerophon
- name: TransRate
  role: Assesses assembly quality after TPM and CD-HIT-EST filtering and applies the
    final TransRate-C chimera-removal filter
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:c4b24f4280921a8531ad4c9615e6a49c8e193580f464a54d29a67bdc2f925a68
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cd-hit-est-redundant-contig-clustering

## Summary

Cluster and remove redundant, highly identical contigs from a de novo transcriptome assembly with CD-HIT-EST at a default 95% identity cut-off, as the second filtration step in the Bellerophon chimera-removal pipeline after TPM filtering.

## When to use

Use when a de novo transcriptome assembly (e.g. Trinity output in FASTA) contains redundant, highly identical contigs that must be deduplicated as part of the Bellerophon filtration sequence (TPM -> CD-HIT-EST -> TransRate-C).

## When NOT to use

- When a custom filtering order is required — the README states 'Custom order of filtering steps is disabled in this version', so CD-HIT-EST cannot be repositioned within the current pipeline.
- When the goal is removing low-expression contigs — that is the TPM filter's role (cut-off 1.0); CD-HIT-EST targets sequence redundancy, not expression level.
- When ORF-length trimming is acceptable instead — the authors set the default order to TPM then CD-HIT-EST 'as the ORF-length filtering appeared to be removing too much transcripts'.

## Inputs

- TPM-filtered de novo transcriptome assembly in FASTA format (e.g. output of the Bellerophon TPM filter step)
- Assembly FASTA plus paired-end FASTQ reads (All_R1/R2) when run via the full Bellerophon pipeline

## Outputs

- Deduplicated (redundant contig-removed) transcriptome assembly FASTA
- Updated TransRate/BUSCO quality scores for the filtered assembly in the Bellerophon results report

## How to apply

Within Bellerophon, CD-HIT-EST is executed after the TPM filter (default cut-off 1) to cluster redundant contigs using a default identity cut-off of 0.95, deliberately more lenient than CD-HIT-EST's own 0.9 default so that fewer near-identical transcripts are merged away. The step takes the TPM-filtered assembly as input and outputs the deduplicated assembly, which is then passed to TransRate-C for final chimera removal. In the reported benchmark, the CD-HIT-EST step removed 600 (24.5%) of the chimeras still present after TPM filtering. Note the trade-off: a CD-HIT-EST-only filter achieved the highest TransRate score (0.319, Figure 2a experiment 3) but showed the smallest improvement in segmented transcripts, leaving 12,096 segmented contigs, so CD-HIT-EST alone is not sufficient for chimera cleanup.

## Related tools

- **CD-HIT-EST** (Clusters and removes highly identical/redundant contigs at a default 95% identity cut-off as the second filtration step of Bellerophon)
- **Bellerophon** (Pipeline that orchestrates the CD-HIT-EST step after TPM filtering and before TransRate-C, and generates the results report) — github.com/JesseKerkvliet/Bellerophon
- **TransRate** (Assesses assembly quality after TPM and CD-HIT-EST filtering and applies the final TransRate-C chimera-removal filter)

## Examples

```
python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- The report confirms CD-HIT-EST ran with the Bellerophon default 95% identity cut-off (more lenient than the tool's 0.9 default) after TPM filtering.
- In the H. subflexa benchmark, the CD-HIT-EST step removed ~24.5% of chimeras remaining after TPM filtering; a large deviation warrants checking the identity threshold.
- A CD-HIT-EST-only filtration should score ≈0.319 under TransRate (Figure 2a, experiment 3); use as a sanity anchor when reproducing the filter comparison.
- The number of segmented contigs should not be expected to drop dramatically from CD-HIT-EST alone — the reported CD-HIT-EST-only condition left 12,096 segmented contigs.
- The filtered assembly must be consumable by the downstream TransRate-C step without format errors, and BUSCO/TransRate-Q should run successfully on it.

## Limitations

- No tool version numbers are stated for CD-HIT-EST (nor for TransRate, BUSCO, or Trinity) in the article.
- Custom ordering of filtering steps is disabled in the current version of Bellerophon, so CD-HIT-EST cannot be moved before TPM filtering.
- The CD-HIT-EST-only filter left the smallest improvement in segmented transcripts (12,096 segmented contigs remained), indicating it is insufficient alone for chimera removal.
- Bellerophon requires the recommended Vagrant VM or conda environment because of backwards-compatibility problems with TransRate, which may affect the CD-HIT-EST step's surroundings.

## Evidence

- [methods] Bellerophon then uses CDHIT-EST to filter redundant contigs, using a default cut-off of 95% identity.: [evidence span withheld — anchor status: not_found]
- [results] the CD‐HIT‐EST filtering step removed 600 (24.5%) of the chimeras still present in the assembly at this stage: "the CD‐HIT‐EST filtering step removed 600 (24.5%) of the chimeras still present in the assembly at this stage"
- [results] The filter with the highest resulting TransRate score was the filter with only CD‐HIT‐EST applied: "The filter with the highest resulting TransRate score was the filter with only CD‐HIT‐EST applied (Figure 2a, experiment number 3), resulting in a TransRate score of 0.319."
- [results] we set the default order to be (1) TPM filtering and (2) CD‐HIT‐EST (filtering order 12 in Figure 2): "we set the default order to be (1) TPM filtering and (2) CD‐HIT‐EST (filtering order 12 in Figure 2), as the ORF‐length filtering appeared to be removing too much transcripts."
- [readme] Custom order of filtering steps is disabled in this version: "Custom order of filtering steps is disabled in this version"
