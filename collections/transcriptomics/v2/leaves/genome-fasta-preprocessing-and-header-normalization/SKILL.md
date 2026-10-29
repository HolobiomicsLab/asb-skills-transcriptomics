---
name: genome-fasta-preprocessing-and-header-normalization
description: Use when a plant reference genome FASTA must be fed into the tncRNA Toolkit
  or any Bowtie-based small-RNA pipeline that requires 'chr'-prefixed headers, masked
  tRNA loci, and an index under lib/indexes/<species>.
when_to_use_negative:
- The genome FASTA headers already follow the '>chr[Num]'/'>chrMt'/'>chrPt' convention
  and a compatible Bowtie index exists in lib/indexes/<species> — re-running normalization
  and index build is redundant
- The downstream analysis is generic genome annotation or alignment that must retain
  unmasked tRNA loci in the reference
- The analysis targets non-plant or organelle-only workflows where the toolkit's nuclear-vs-organellar
  ('chrMt'/'chrPt') separation does not apply
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3489
tools:
- name: tncRNA Toolkit
  role: Driver script tncRNAs.py performs header-dependent index building and artificial-genome
    creation; invoked as 'python3 tncRNAs.py -g <genome fasta> -s <species name>'
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: bedtools
  role: maskfasta (v2.29) masks the filtered tRNA regions plus 50 bp 3'/5' flanks
    in the reference genome before artificial chromosomes are concatenated
- name: Bowtie
  role: bowtie-build (v1.3) builds the index for the artificial genome, writing *.ebwt
    files under lib/indexes/<species name>
- name: tRNAscan-SE (v2.0.6)
  role: Predicts nuclear tRNAs (eukaryote model) and organellar tRNAs ('-O' option)
    to define which regions are masked and added as artificial chromosomes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:c7bc637596e64da919bc3a19efc350397c85c71d62b0ca301d7699b6b8306e3d
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
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# genome-fasta-preprocessing-and-header-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocesses a reference genome FASTA for the tncRNA Toolkit pipeline: normalizes FASTA headers to the '>chr[Num]' convention (with '>chrMt' and '>chrPt' for organellar sequences), then builds the Bowtie index after masking tRNA genes plus 50 nt flanks and appending tRNA-derived artificial chromosomes.

## When to use

Use when a plant reference genome FASTA must be fed into the tncRNA Toolkit or any Bowtie-based small-RNA pipeline that requires 'chr'-prefixed headers, masked tRNA loci, and an index under lib/indexes/<species>.

## When NOT to use

- The genome FASTA headers already follow the '>chr[Num]'/'>chrMt'/'>chrPt' convention and a compatible Bowtie index exists in lib/indexes/<species> — re-running normalization and index build is redundant
- The downstream analysis is generic genome annotation or alignment that must retain unmasked tRNA loci in the reference
- The analysis targets non-plant or organelle-only workflows where the toolkit's nuclear-vs-organellar ('chrMt'/'chrPt') separation does not apply

## Inputs

- Reference genome FASTA (nuclear plus organellar assemblies, e.g. TAIR10.1, SL4.0/SL3.0)
- tRNAscan-SE predicted tRNA set (pseudogenes and score < 50 removed)
- Filtered tRNA FASTA files: mature tRNA with CCA added, 50 bp 5' leader flanks, 50 bp 3' trailer flanks
- BED file of tRNA regions with 50 bp 3'/5' flanks for bedtools maskfasta

## Outputs

- Masked reference genome FASTA (tRNA genes + 50 nt flanks masked)
- Artificial genome FASTA containing masked chromosomes plus tRNA-derived artificial chromosomes ('chr'-prefixed, chrMt/chrPt for organellar)
- Bowtie v1.3 index files (artificial_genome.*.ebwt) under lib/indexes/<species name>

## How to apply

First rewrite every FASTA header so chromosome entries begin with '>chr[Num]' (e.g. >chr1, >chr2) and mitochondrial/plastid entries use '>chrMt' and '>chrPt', since the toolkit scripts automate on this naming. Predict tRNAs with tRNAscan-SE v2.0.6 (eukaryote model for nuclear, '-O' for organellar), remove pseudogenes and tRNAs scoring below 50, and prepare mature tRNA (intron-removed, CCA-added), 50 bp 5' leader and 50 bp 3' trailer FASTA files. Mask the retained tRNA regions plus their 50 bp flanks in the reference genome using bedtools v2.29 maskfasta, then concatenate the three tRNA FASTA files onto the masked genome as additional chromosomes to create the artificial genome. Run 'python3 tncRNAs.py -g <genome fasta> -s <species name>' to build the Bowtie v1.3 index with bowtie-build; the index and auxiliary files are placed automatically in lib/indexes/<species name>. The rationale is that masking plus exclusive artificial-chromosome mapping prevents ambiguous reads from non-tRNA regions being misclassified as tncRNAs.

## Related tools

- **tncRNA Toolkit** (Driver script tncRNAs.py performs header-dependent index building and artificial-genome creation; invoked as 'python3 tncRNAs.py -g <genome fasta> -s <species name>') — https://github.com/skbinfo/tncRNA-Toolkit
- **bedtools** (maskfasta (v2.29) masks the filtered tRNA regions plus 50 bp 3'/5' flanks in the reference genome before artificial chromosomes are concatenated)
- **Bowtie** (bowtie-build (v1.3) builds the index for the artificial genome, writing *.ebwt files under lib/indexes/<species name>)
- **tRNAscan-SE (v2.0.6)** (Predicts nuclear tRNAs (eukaryote model) and organellar tRNAs ('-O' option) to define which regions are masked and added as artificial chromosomes)

## Examples

```
python3 tncRNAs.py -g genome.fa -s species_name
```

## Evaluation signals

- All FASTA headers in the processed genome begin with '>chr[Num]', with organellar contigs named '>chrMt' and '>chrPt'
- Bowtie index files for the artificial genome (artificial_genome.*.ebwt) exist under lib/indexes/<species name>
- The artificial genome FASTA contains the masked reference chromosomes plus the additional tRNA-derived chromosomes (mature tRNA, 50 bp leader, 50 bp trailer entries)
- Masked regions correspond exactly to filtered tRNA genes (pseudogenes and tRNAscan-SE score < 50 removed) extended by 50 nt on both 3' and 5' sides
- Aligning a probe read with bowtie '-v2 --best' returns a hit only against the artificial chromosomes for genuine tncRNA sequences

## Limitations

- Header convention is rigid: the toolkit requires '>chr'-prefixed names and chrMt/chrPt for organellar contigs, otherwise automation of nuclear vs organellar separation fails
- The artificial genome is species-specific — a new Bowtie index must be built per species under lib/indexes/<species name>
- Allowing mismatches during downstream alignment can inflate false negatives via base misidentification, which is why masking/exclusive artificial-chromosome classification is used

## Evidence

- [methods] Masking of tRNA regions plus flanks with bedtools: [evidence span withheld — anchor status: not_found]
