---
name: isoform-rich-contig-group-selection
description: Use when benchmarking a transcriptome chimera-removal pipeline (e.g.
  Bellerophon filter orders) and you need a targeted, chimera-rich test set built
  from isoform-rich contig groups rather than the whole assembly.
when_to_use_negative:
- The assembly has no Trinity-predicted isoform structure or is not a de novo transcriptome
  (e.g. genome-guided or reference-based transcripts), since the contig-group selection
  criterion depends on Trinity isoform groups
- You need whole-assembly chimera statistics rather than a focused, isoform-rich benchmark
  subset
- You require a reference-genome-based quality assessment; this skill is designed
  for the reference-free de novo setting where no reference genome is available
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: Bellerophon
  role: Pipeline under test; run with a custom filter order (TransRate-C before RSEM-based
    TPM filtering) on the isoform-rich benchmark set
  repo: github.com/JesseKerkvliet/Bellerophon
- name: Trinity
  role: De novo assembler whose predicted isoform groups define the contig groups
    selected for benchmarking
- name: BLASTX
  role: Marks chimeric isoforms by BLASTX against the NCBI NR database (E-value cut-off
    1e-4), re-run after each filtering step
- name: TransRate
  role: Provides the TransRate-C filtering step evaluated in the variant ordering
- name: CD-HIT-EST
  role: Redundancy filter (95% identity default) in the default order used as the
    comparison baseline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:fc35dcaf9289d44a7958b809731f065f5822211c40da74488d8c8fc9e3f1bf84
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTX
  - Bellerophon
  - CD-HIT-EST
  - TransRate
  - Trinity
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# isoform-rich-contig-group-selection

## Summary

A benchmark-design skill for evaluating chimera-removal filters on de novo transcriptomes: select the contig groups with the highest number of Trinity-predicted isoforms from an assembly (e.g. the H. subflexa pheromone gland assembly), mark chimeras with BLASTX against NCBI NR, and count how many chimeric isoforms survive each filtering configuration.

## When to use

Use when benchmarking a transcriptome chimera-removal pipeline (e.g. Bellerophon filter orders) and you need a targeted, chimera-rich test set built from isoform-rich contig groups rather than the whole assembly.

## When NOT to use

- The assembly has no Trinity-predicted isoform structure or is not a de novo transcriptome (e.g. genome-guided or reference-based transcripts), since the contig-group selection criterion depends on Trinity isoform groups
- You need whole-assembly chimera statistics rather than a focused, isoform-rich benchmark subset
- You require a reference-genome-based quality assessment; this skill is designed for the reference-free de novo setting where no reference genome is available

## Inputs

- De novo transcriptome assembly (Trinity FASTA, e.g. H. subflexa pheromone gland assembly) with Trinity-predicted isoform grouping
- Illumina RNA-Seq reads (e.g. H. subflexa pheromone gland reads, NCBI BioProject PRJNA527373: SRR8735410, SRR8735411, SRR8735412)
- NCBI nonredundant protein database (NR) for BLASTX chimera marking

## Outputs

- Ranked selection of the 10 contig groups with the most Trinity-predicted isoforms
- BLASTX-based chimera annotations (contigs with two non-overlapping NR hits to different genes, E-value <= 1e-4)
- Per-filtering-step counts of remaining chimeric isoforms (e.g. 74 -> 18 after TransRate-C-before-RSEM order, 56 chimeras removed)
- Comparison table of variant vs. default filter orders

## How to apply

From the de novo Trinity assembly (e.g. the H. subflexa pheromone gland assembly), select the 10 contig groups containing the highest number of Trinity-predicted isoforms. Mark chimeric isoforms by running BLASTX against the NCBI NR protein database with an E-value cut-off of 1e-4, flagging a contig as chimeric when it has two non-overlapping matches to different genes (this yielded 74 chimeric isoforms in the benchmark set). Then run the filtering variant under test — for example, the Bellerophon pipeline with a custom order placing TransRate-C filtering before the RSEM-based TPM filtering step, using the Illumina pheromone gland RNA-Seq reads — and after each filtering step re-run the BLASTX marking and count remaining chimeras. Compare against the default order (TPM first, CD-HIT-EST at 95% identity second) to justify why TPM filtering is placed first by default; in the published benchmark the variant order reduced chimeras from 74 to 18 (56 removed).

## Related tools

- **Bellerophon** (Pipeline under test; run with a custom filter order (TransRate-C before RSEM-based TPM filtering) on the isoform-rich benchmark set) — github.com/JesseKerkvliet/Bellerophon
- **Trinity** (De novo assembler whose predicted isoform groups define the contig groups selected for benchmarking)
- **BLASTX** (Marks chimeric isoforms by BLASTX against the NCBI NR database (E-value cut-off 1e-4), re-run after each filtering step)
- **TransRate** (Provides the TransRate-C filtering step evaluated in the variant ordering)
- **CD-HIT-EST** (Redundancy filter (95% identity default) in the default order used as the comparison baseline)

## Examples

```
python Bellerophon.py --assembly Hsubflexa_trinity.fasta --left SRR8735410_R1.fastq --right SRR8735410_R2.fastq --outdir /vagrant/testrun
```

## Evaluation signals

- BLASTX marking with E-value cut-off 1e-4 identifies exactly 74 chimeric isoforms across the 10 highest-isoform contig groups before filtering
- After the TransRate-C-before-RSEM variant order, the output table shows 18 chimeric isoforms remaining (56 removed), matching the paper's reported benchmark outcome
- Chimera marking is re-applied after every filtering step, so counts are available per step (TPM, CD-HIT-EST, TransRate-C), not only at the end
- The variant-order result is compared against the default order (TPM first, CD-HIT-EST 95% identity second) to confirm why TPM filtering is placed first by default

## Limitations

- Custom order of filtering steps is disabled in the current released Bellerophon version, so the variant ordering requires a modified pipeline
- Bellerophon may erroneously remove contigs from alternative splicing that are misclassified as chimeras (uneven read coverage along a transcript can look chimeric to TransRate)
- Chimera marking depends on informative BLASTX hits against NR; contigs without two non-overlapping matches to different genes cannot be flagged

## Evidence

- [other] Select the 10 contig groups with the highest number of Trinity-predicted isoforms from the H. subflexa pheromone gland assembly: [evidence span withheld — anchor status: not_found]
- [methods] Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes.: "Contigs were identified as chimeric when a sequence contained two matches that did not overlap in their mapping region and had hits with different genes."
- [other] Running TransRate-C before running RSEM decreased the number of chimeric isoforms in the isoform-rich benchmark set from 74 to 18, removing 56 chimeras.: [evidence span withheld — anchor status: not_found]
- [results] we set the default order to be (1) TPM filtering and (2) CD‐HIT‐EST: "we set the default order to be (1) TPM filtering and (2) CD‐HIT‐EST (filtering order 12 in Figure 2), as the ORF‐length filtering appeared to be removing too much transcripts."
- [readme] Custom order of filtering steps is disabled in this version: "Custom order of filtering steps is disabled in this version"
