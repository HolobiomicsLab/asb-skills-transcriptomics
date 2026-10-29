---
name: transcriptome-read-simulation
description: 'Use when a de novo transcriptome pipeline (e.g., chimera removal) must
  be benchmarked against a known ground truth: a reference transcriptome and expression
  profile are available to simulate reads from.'
when_to_use_negative:
- The input is real RNA-Seq data with no known reference transcriptome — no ground
  truth exists to score chimera removal against
- A reference genome is available, making alignment-based assessment of the assembly
  possible without simulation
- The goal is to benchmark expression quantification accuracy rather than chimera/content
  filtering of a de novo assembly
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Polyester
  role: Simulates paired-end RNA-Seq reads from the reference transcriptome using
    fpkm_to_counts, create_read_numbers, and simulate_experiment_countmat with the
    H. subflexa expression profile
- name: Trinity
  role: Assembles the simulated reads de novo with default parameters to produce the
    ground-truth assembly
- name: BLASTn
  role: Classifies assembled contigs as chimeric or correct against the reference
    transcriptome (90% identity, E-value 10)
- name: Bellerophon
  role: Consumes the simulated-and-labeled Trinity assembly to benchmark chimera removal
    (TPM filter cutoff 1, CD-HIT-EST 95% identity, TransRate filtering step)
  repo: github.com/JesseKerkvliet/Bellerophon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.5571
    title: The Bellerophon pipeline, improving de novo transcriptomes and removing
      chimeras
  source_content_hash: sha256:39078c0910954fc8d3ca948a4f34594535881f6a2bc12fc42712ca463d98cd90
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTn
  - Bellerophon
  - CD-HIT-EST
  - Polyester
  - TransRate
  - Trinity
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# transcriptome-read-simulation

## Summary

Simulate paired-end RNA-Seq reads from a known reference transcriptome with Polyester so that de novo assembly and chimera-removal pipelines can be benchmarked against a ground truth, as done for the Bellerophon pipeline with 3,000 D. melanogaster transcripts.

## When to use

Use when a de novo transcriptome pipeline (e.g., chimera removal) must be benchmarked against a known ground truth: a reference transcriptome and expression profile are available to simulate reads from.

## When NOT to use

- The input is real RNA-Seq data with no known reference transcriptome — no ground truth exists to score chimera removal against
- A reference genome is available, making alignment-based assessment of the assembly possible without simulation
- The goal is to benchmark expression quantification accuracy rather than chimera/content filtering of a de novo assembly

## Inputs

- Reference transcriptome FASTA: 3,000 randomly selected D. melanogaster transcripts from NCBI RefSeq GCF_000001215.4
- Expression profile (FPKM values) of contigs from the H. subflexa female pheromone gland assembly
- Polyester (read simulation tool)

## Outputs

- Three paired-end FASTQ read sets of 20,370,192, 19,995,866, and 20,045,180 reads
- Trinity de novo assembly (contigs with >=5 read support) labeled by BLASTn as chimeric (295 contigs) or correct (3,283 contigs)
- Filtered transcriptome with per-filter removal counts (e.g., 136/295 chimeras and 575/3,283 correct sequences removed by Bellerophon)

## How to apply

Randomly select 3,000 transcripts from the D. melanogaster reference genome annotation (NCBI RefSeq GCF_000001215.4) to serve as the reference transcriptome. Define the expression profile by taking the expression values (FPKM) of contigs from the H. subflexa female pheromone gland assembly as input to Polyester's 'fpkm_to_counts' and 'create_read_numbers' functions. Generate three paired-end read sets — 20,370,192, 19,995,866, and 20,045,180 reads — using Polyester's 'simulate_experiment_countmat' function. Assemble the simulated reads de novo with Trinity (default parameters) and discard assembled contigs supported by fewer than 5 reads. Classify contigs as chimeric (matching more than one reference transcript) or correct (matching exactly one) by BLASTn against the reference transcriptome at 90% percent identity and E-value 10; this labeled assembly then serves as the ground truth for evaluating downstream filters such as Bellerophon (TPM cutoff 1, CD-HIT-EST 95% identity, TransRate filtering).

## Related tools

- **Polyester** (Simulates paired-end RNA-Seq reads from the reference transcriptome using fpkm_to_counts, create_read_numbers, and simulate_experiment_countmat with the H. subflexa expression profile)
- **Trinity** (Assembles the simulated reads de novo with default parameters to produce the ground-truth assembly)
- **BLASTn** (Classifies assembled contigs as chimeric or correct against the reference transcriptome (90% identity, E-value 10))
- **Bellerophon** (Consumes the simulated-and-labeled Trinity assembly to benchmark chimera removal (TPM filter cutoff 1, CD-HIT-EST 95% identity, TransRate filtering step)) — github.com/JesseKerkvliet/Bellerophon

## Examples

```
from polyester import fpkm_to_counts, create_read_numbers, simulate_experiment_countmat
```

## Evaluation signals

- The three simulated read sets contain 20,370,192, 19,995,866, and 20,045,180 reads respectively
- BLASTn classification (90% identity, E-value 10) of the Trinity assembly yields 295 chimeras and 3,283 correct sequences
- Bellerophon removes 136 of the 295 chimeras (46.1%) and 575 of the 3,283 correct sequences (17.5%)
- Every assembled contig matching more than one reference transcript is labeled chimeric; contigs matching exactly one are labeled correct
- Assembled contigs supported by fewer than 5 reads are excluded from the benchmark

## Limitations

- No version numbers were stated for Polyester, Trinity, BLASTn, TransRate, or CD-HIT-EST in the methods
- Chimera/correct labels depend entirely on the chosen BLASTn thresholds (90% percent identity, E-value 10), so reclassification can shift the 295/3,283 ground-truth counts
- Bellerophon applied to such assemblies may erroneously remove contigs from alternative splicing that are misclassified as chimeras

## Evidence

- [methods] a simulated RNA‐Seq experiment obtained through the tool polyester (Frazee, Jaffe, Langmead, & Leek, 2015) and a reference transcriptome of 3,000 transcripts expressed by Drosophila melanogaster: "a simulated RNA‐Seq experiment obtained through the tool polyester (Frazee, Jaffe, Langmead, & Leek, 2015) and a reference transcriptome of 3,000 transcripts expressed by Drosophila melanogaster"
- [methods] from the D. melanogaster reference genome (NCBI RefSeq GCF_000001215.4).: "from the D. melanogaster reference genome (NCBI RefSeq GCF_000001215.4)."
- [results] we used 3,000 transcripts expressed by D. melanogaster as a reference to simulate RNA‐Seq reads.: "we used 3,000 transcripts expressed by D. melanogaster as a reference to simulate RNA‐Seq reads."
- [readme] Bellerophon will make an output directory in the /vagrant/ directory.: "Note that Bellerophon will make an output directory in the /vagrant/ directory."
