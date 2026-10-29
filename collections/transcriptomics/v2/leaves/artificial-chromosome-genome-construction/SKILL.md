---
name: artificial-chromosome-genome-construction
description: Use when small RNA-seq reads must be classified as tRNA-derived without
  ambiguity from non-tRNA loci, requiring a modified reference genome where tRNA regions
  plus flanks are separated into distinct artificial chromosomes.
when_to_use_negative:
- Input is not a reference genome plus tRNA annotations (e.g. already-aligned BAM
  or a feature count table) — masking and artificial chromosome construction does
  not apply
- The goal is generic small RNA annotation against a standard reference, where exclusive-to-artificial-chromosome
  mapping is unnecessary
- Tools must run exclusively on human datasets pre-trained pipelines (e.g. tDRmapper,
  MINTmap, tRF2Cancer) rather than de novo genome construction for plant species
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0085
tools:
- name: bedtools
  role: maskfasta script (v2.29) is used to mask the filtered tRNA regions plus 50
    bp 3' and 5' flanks in the reference genome
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Bowtie
  role: bowtie-build (bowtie v1.3) builds the Bowtie index for the artificial genome,
    which is then used for error-tolerant uniquely mapping read alignment
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: tRNAscan-SE (v2.0.6)
  role: predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNAs that
    define the regions masked and appended as artificial chromosomes
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:562254dc3575b88962b1bdcaabbe7eb43dec4f897935394c14344b13d6e2078d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - Bowtie
  - bedtools
  - tRNAscan-SE (v2.0.6)
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# artificial-chromosome-genome-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds an artificial genome for unambiguous tncRNA identification by masking genuine tRNA gene regions plus 50 bp flanks in the reference genome (bedtools maskfasta) and appending mature tRNA, 5' leader, and 3' trailer FASTA sequences as artificial chromosomes, then indexing with bowtie-build.

## When to use

Use when small RNA-seq reads must be classified as tRNA-derived without ambiguity from non-tRNA loci, requiring a modified reference genome where tRNA regions plus flanks are separated into distinct artificial chromosomes.

## When NOT to use

- Input is not a reference genome plus tRNA annotations (e.g. already-aligned BAM or a feature count table) — masking and artificial chromosome construction does not apply
- The goal is generic small RNA annotation against a standard reference, where exclusive-to-artificial-chromosome mapping is unnecessary
- Tools must run exclusively on human datasets pre-trained pipelines (e.g. tDRmapper, MINTmap, tRF2Cancer) rather than de novo genome construction for plant species

## Inputs

- Reference genome FASTA (headers beginning '>chr[Num]', '>chrMt', '>chrPt')
- tRNAscan-SE predicted tRNA set (nuclear via eukaryote model, organellar via '-O')
- Filtered genuine tRNAs (pseudogenes removed, Covels score >= 50) as mature tRNA, 50 bp 5' leader, and 50 bp 3' trailer FASTA files

## Outputs

- Artificial genome FASTA (masked reference chromosomes plus tRNA-derived artificial chromosomes)
- Bowtie index files under lib/indexes/<species name> (artificial_genome.*.ebwt)

## How to apply

First filter the tRNAscan-SE predicted tRNA set by removing pseudogenes and any tRNA with a Covels score below 50 (eukaryote model for nuclear tRNAs, '-O' option for organellar). Prepare three FASTA files from the filtered tRNAs: mature tRNA (intron-removed with CCA added at the 3' terminus), 50 bp 5' leader flank, and 50 bp 3' trailer flank. Using bedtools (v2.29) maskfasta, mask the tRNA regions plus 50 bp flanks on both 3' and 5' sides in the reference genome, then concatenate the three tRNA FASTA files onto the masked genome as additional chromosomes to create the artificial genome. Build the Bowtie index with bowtie-build (bowtie v1.3), writing index files under lib/indexes/<species name>; genome FASTA headers must start with '>chr[Num]' (e.g. >chr1, >chrMt, >chrPt) so scripts can separate nuclear and organellar regions. Reads are later classified as tncRNAs only if they map exclusively to these artificial chromosomes, which resolves ambiguous mapping of tRNA-derived reads to non-tRNA genomic regions.

## Related tools

- **bedtools** (maskfasta script (v2.29) is used to mask the filtered tRNA regions plus 50 bp 3' and 5' flanks in the reference genome) — https://github.com/skbinfo/tncRNA-Toolkit
- **Bowtie** (bowtie-build (bowtie v1.3) builds the Bowtie index for the artificial genome, which is then used for error-tolerant uniquely mapping read alignment) — https://github.com/skbinfo/tncRNA-Toolkit
- **tRNAscan-SE (v2.0.6)** (predicts nuclear (eukaryotes model) and organellar ('-O' option) tRNAs that define the regions masked and appended as artificial chromosomes) — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
python3 tncRNAs.py -g <genome fasta> -s <species name>
```

## Evaluation signals

- Bowtie index files for the artificial genome exist (artificial_genome.*.ebwt) under lib/indexes/<species name>
- The artificial genome FASTA contains the masked reference chromosomes plus the additional tRNA-derived artificial chromosomes
- All genome FASTA headers follow the '>chr[Num]' convention (nuclear >chr1, >chr2...; organellar >chrMt, >chrPt)
- Downstream classification yields tncRNAs only among reads that mapped exclusively to the artificial chromosomes, avoiding ambiguous reads from non-tRNA regions

## Limitations

- Multi-mapped reads are discarded if alignment occurs greater than 50 times (bowtie1 '--norc -v 0 -m 50'), so repetitive tncRNA loci may be lost
- Allowing a single mismatch to overcome sequencing errors may lead to base misidentification and inflation of false negatives
- The default mismatch/gap value is 0; values greater than 3 are not recommended for the Bowtie -v option

## Evidence

- [results] Mask the filtered tRNA regions plus 50 bp flanks on both the 3' and 5' sides in the reference genome using the maskfasta script of bedtools (v2.29).: [evidence span withheld — anchor status: not_found]
- [results] we created an artificial genome by masking the genuine tRNA gene region with 50 nt upstream and downstream in the genome (masked genome) and adding them as artificial chromosomes: "we created an artificial genome by masking the genuine tRNA gene region with 50 nt upstream and downstream in the genome (masked genome) and adding them as artificial chromosomes"
- [methods] To avoid ambiguous reads from non-tRNA regions, we created an artificial genome by masking the genuine tRNA gene region with 50 nt upstream and downstream: "To avoid ambiguous reads from non-tRNA regions, we created an artificial genome by masking the genuine tRNA gene region with 50 nt upstream and downstream"
- [readme] Genome fasta header should be start with '>chr[Num]', for example >chr1, >chr2 so on. Mitochondrial and plastid fasta headers also should be as >chrMt & >chrPt respectively.: "Genome fasta header should be start with '>chr[Num]', for example >chr1, >chr2 so on. Mitochondrial and plastid fasta headers also should be as >chrMt & >chrPt respectively."
- [methods] The tRNA pseudogenes and tRNAs with a score less than 50 were eliminated: "The tRNA pseudogenes and tRNAs with a score less than 50 were eliminated"
