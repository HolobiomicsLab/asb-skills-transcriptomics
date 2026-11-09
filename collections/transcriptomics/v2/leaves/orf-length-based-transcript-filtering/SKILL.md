---
name: orf-length-based-transcript-filtering
description: Use when a de novo transcriptome assembly (e.g., Trinity FASTA contigs)
  must be filtered for likely non-coding or fragmented transcripts by predicted ORF
  length, as part of chimera-removal or assembly-cleanup filtering.
when_to_use_negative:
- 'When maximizing TransRate score is the only goal: the CD-HIT-EST-only filter scored
  highest (0.319) and ORF-length filtering removed too many transcripts, so it is
  suboptimal for that objective'
- 'When the current Bellerophon release is required with custom filter ordering: ''Custom
  order of filtering steps is disabled in this version'''
- When the input is already a quality-filtered or reference-guided transcript set
  where non-coding fragments were removed by other means
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Bellerophon
  role: Pipeline that implements the ORF-length filter (default 50 aa or 100 aa) alongside
    TPM and CD-HIT-EST filters and scores assemblies with TransRate
  repo: github.com/JesseKerkvliet/Bellerophon
- name: TransRate
  role: Reference-free quality assessment tool used to score the assembly before and
    after ORF-length filtering to judge whether the filter improved quality
- name: CD-HIT-EST
  role: Companion filter removing highly identical (95% identity) contigs; the CD-HIT-only
    variant outperformed the variant including ORF-length filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:08a4e9362dc8ed6f380106ff5136b4d64491703a6f67729333315ae4cbd24e1e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# orf-length-based-transcript-filtering

## Summary

Removes transcript contigs from a de novo assembly whose predicted open reading frame (ORF) is too short, using community-default thresholds of 50 aa or 100 aa, as one of the three filter components (CD-HIT-EST, TPM, ORF length) explored in the Bellerophon pipeline.

## When to use

Use when a de novo transcriptome assembly (e.g., Trinity FASTA contigs) must be filtered for likely non-coding or fragmented transcripts by predicted ORF length, as part of chimera-removal or assembly-cleanup filtering.

## When NOT to use

- When maximizing TransRate score is the only goal: the CD-HIT-EST-only filter scored highest (0.319) and ORF-length filtering removed too many transcripts, so it is suboptimal for that objective
- When the current Bellerophon release is required with custom filter ordering: 'Custom order of filtering steps is disabled in this version'
- When the input is already a quality-filtered or reference-guided transcript set where non-coding fragments were removed by other means

## Inputs

- De novo transcriptome assembly in FASTA format (e.g., Trinity contigs from H. subflexa pheromone gland RNA-Seq or simulated D. melanogaster RNA-Seq)
- Illumina paired-end RNA-Seq reads (FASTQ) used by TransRate for quality assessment
- ORF-length threshold (community default 50 aa or 100 aa)

## Outputs

- Filtered FASTA assembly with contigs shorter than the ORF-length cut-off removed
- TransRate quality score of the filtered assembly (e.g., 0.316 for CD-HIT+TPM+ORF-length in the reported comparison)
- Contig counts before/after filtering (e.g., 16,057 remaining contigs of 93,659; 3,293 segmented contigs)

## How to apply

Within the Bellerophon workflow, first establish a ground quality score of the unfiltered assembly with TransRate (plus BUSCO for completeness context) using the assembly and the sequencing reads. Then apply the ORF-length filter with the community-default threshold of 50 aa (or 100 aa), optionally in combination with the TPM filter (cut-off 1.0) and CD-HIT-EST (95% identity). Note that the Bellerophon authors found ORF-length filtering 'appeared to be removing too much transcripts': the combined CD-HIT + TPM + ORF-length filter scored 0.316 on TransRate (second-best), removing 77,602 (82.9%) contigs and leaving 16,057 contigs, while the CD-HIT-EST-only filter scored highest at 0.319. After filtering, re-score the filtered assembly with TransRate-Q and BUSCO and compare against the benchmark conditions to decide whether ORF-length filtering improves the assembly quality score.

## Related tools

- **Bellerophon** (Pipeline that implements the ORF-length filter (default 50 aa or 100 aa) alongside TPM and CD-HIT-EST filters and scores assemblies with TransRate) — github.com/JesseKerkvliet/Bellerophon
- **TransRate** (Reference-free quality assessment tool used to score the assembly before and after ORF-length filtering to judge whether the filter improved quality)
- **CD-HIT-EST** (Companion filter removing highly identical (95% identity) contigs; the CD-HIT-only variant outperformed the variant including ORF-length filtering)

## Examples

```
python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- The ORF-length cut-off equals the community default of 50 aa or 100 aa
- For the combined CD-HIT+TPM+ORF-length condition, ~82.9% of contigs are removed, leaving ≈16,057 contigs
- TransRate score of the combined filter ≈0.316, second-best versus CD-HIT-EST-only (≈0.319), consistent with Figure 2a ordering
- Filtered assembly still yields a measurable BUSCO completeness using the Insecta odb9 ortholog set
- Per-condition TransRate scores are collated in filter_order_scores.csv for comparison across filter orders

## Limitations

- ORF-length filtering 'appeared to be removing too much transcripts', so it is not part of the default Bellerophon order (TPM then CD-HIT-EST, filtering order 12 in Figure 2)
- No tool version numbers are stated for the filtering tools (TransRate, CD-HIT-EST, Trinity, BUSCO), risking compatibility issues; the Vagrant VM is recommended to avoid TransRate backwards-compatibility problems
- Custom ordering of filtering steps (including ORF-length placement) is disabled in the current software version
- Bellerophon may erroneously remove contigs from alternative splicing that TransRate misclassifies as chimeric if expression is uneven along the sequence

## Evidence

- [methods] Default thresholds were determined using community defaults for TPM (1.0) and ORF length (50 aa or 100 aa).: "Default thresholds were determined using community defaults for TPM (1.0) and ORF length (50 aa or 100 aa)."
- [results] Using the three filter components CD-HIT-EST, TPM, and ORF length, an optimal filtering method was designed.: "Using the three filter components CD‐HIT‐EST, TPM, and ORF length, an optimal filtering method was designed."
- [results] this filter removed 77,602 (82.9%) contigs, leaving the set with only 16,057 contigs.: "this filter removed 77,602 (82.9%) contigs, leaving the set with only 16,057 contigs."
- [results] as the ORF-length filtering appeared to be removing too much transcripts.: "we set the default order to be (1) TPM filtering and (2) CD‐HIT‐EST (filtering order 12 in Figure 2), as the ORF‐length filtering appeared to be removing too much transcripts."
- [results] The highest resulting TransRate score was the filter with only CD-HIT-EST applied.: "The filter with the highest resulting TransRate score was the filter with only CD‐HIT‐EST applied (Figure 2a, experiment number 3), resulting in a TransRate score of 0.319."
- [abstract] The developed method is incorporated into a pipeline, which we named Bellerophon, that is broadly applicable and easy to use.: "The developed method is incorporated into a pipeline, which we named Bellerophon, that is broadly applicable and easy to use."
- [readme] python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun: "python Bellerophon.py --assembly Demo.fasta --left All_R1_1000.fastq --right All_R2_1000.fastq --outdir /vagrant/testrun"
