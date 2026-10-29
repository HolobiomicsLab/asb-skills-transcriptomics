---
name: isoform-assembly-benchmark-reproduction
description: 'Use when reproducing the Table 3 ''1'' vs ''1-fixed'' comparison on
  real SPIso-seq contigs: you have a cloudrnaSPAdes contig FASTA plus a barcode-to-gene
  assignment table and must measure how many ambiguously assigned contigs a duplicate-barcode
  filter removes.'
when_to_use_negative:
- Input is already-filtered ('1-fixed') output — reapplying the duplicate-barcode
  filter would double-remove and inflate the removal rate beyond the ~15% Table 3
  difference
- Data are the simulated human data sets or the SPAdes toy data set (first 1000 bp
  of E. coli) rather than the real SPIso-seq contigs the Table 3 fixed rows refer
  to
- The goal is read-level filtering before assembly (e.g. median k-mer coverage filtering
  with spades-read-filter) rather than post-assembly contig filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: cloudrnaSPAdes
  role: Tool whose isoform assembly output (real SPIso-seq contigs FASTA) is the input
    being benchmarked and whose paper defines the barcode-filtering step being reproduced
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: SPAdes
  role: Underlying assembly toolkit; cloudrnaSPAdes is a feature release of SPAdes,
    and its manual describes the output contigs and stand-alone binaries used around
    the assembly
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: Bandage
  role: Optional GUI inspection of the SPAdes assembly graph (GFA format) when tracing
    which contigs carry duplicate-barcode assignments
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:18de7aabc3506d324fc4d67348b760cd786cd494a3c1072fae77ffbdb6ef4749
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - Bandage
  - SPAdes
  - cloudrnaSPAdes
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# isoform-assembly-benchmark-reproduction

## Summary

Reproduces the barcode-based post-assembly filtering step from cloudrnaSPAdes, which removes contigs whose barcode is assigned to a single gene multiple times, and quantifies how much of the ambiguous-assignment fraction this removes (~15% difference between the '1' and '1-fixed' Table 3 rows) on real SPIso-seq contigs assembled by cloudrnaSPAdes.

## When to use

Use when reproducing the Table 3 '1' vs '1-fixed' comparison on real SPIso-seq contigs: you have a cloudrnaSPAdes contig FASTA plus a barcode-to-gene assignment table and must measure how many ambiguously assigned contigs a duplicate-barcode filter removes.

## When NOT to use

- Input is already-filtered ('1-fixed') output — reapplying the duplicate-barcode filter would double-remove and inflate the removal rate beyond the ~15% Table 3 difference
- Data are the simulated human data sets or the SPAdes toy data set (first 1000 bp of E. coli) rather than the real SPIso-seq contigs the Table 3 fixed rows refer to
- The goal is read-level filtering before assembly (e.g. median k-mer coverage filtering with spades-read-filter) rather than post-assembly contig filtering

## Inputs

- Assembled real-data contigs FASTA produced by cloudrnaSPAdes from the SPIso-seq barcoded RNA-seq data
- Barcode-to-gene assignment table for the same SPIso-seq data set (barcode ID, assigned gene, assignment count)

## Outputs

- Filtered FASTA of retained contigs (contigs whose barcode is not assigned to a single gene multiple times)
- barcode_filtering_rates.csv with removed/retained counts and removal rates (percentage) for ambiguous and unique assignments

## How to apply

Load the assembled real-data contigs FASTA produced by cloudrnaSPAdes alongside the barcode-to-gene assignment table for the same SPIso-seq data set. For each contig, classify its barcode assignment as unique (barcode maps to a single gene once) or ambiguous (barcode assigned to a single gene multiple times). Run the paper's barcode-filtering step — remove every contig whose barcode is assigned to a single gene multiple times — and write the retained contigs to a filtered FASTA. Count removed and retained contigs per category and compute the removal percentage separately for ambiguous and unique assignments, writing counts and rates to barcode_filtering_rates.csv. The validation target is reproducing the ~15% filtering difference between the '1' and '1-fixed' rows of Table 3, with unique assignments remaining relatively unchanged; assembly graphs can be inspected with Bandage (GFA output supported) if contig provenance needs checking.

## Related tools

- **cloudrnaSPAdes** (Tool whose isoform assembly output (real SPIso-seq contigs FASTA) is the input being benchmarked and whose paper defines the barcode-filtering step being reproduced) — https://github.com/1dayac/cloudrnaSPAdes-release
- **SPAdes** (Underlying assembly toolkit; cloudrnaSPAdes is a feature release of SPAdes, and its manual describes the output contigs and stand-alone binaries used around the assembly) — https://github.com/1dayac/cloudrnaSPAdes-release
- **Bandage** (Optional GUI inspection of the SPAdes assembly graph (GFA format) when tracing which contigs carry duplicate-barcode assignments) — https://github.com/rrwick/Bandage

## Examples

```
python -c "import pandas as pd; m=pd.read_csv('barcode_gene.tsv',sep='\t'); dup=m[m.duplicated(subset=['barcode','gene'],keep=False)]; print(len(dup)/len(m)*100)"
```

## Evaluation signals

- The ambiguous-assignment removal rate reproduces the ~15% difference between the '1' and '1-fixed' rows of Table 3
- Unique-assignment counts remain relatively unchanged between the '1' and '1-fixed' rows (filter should not remove uniquely assigned contigs)
- barcode_filtering_rates.csv contains counts and percentages for both ambiguous and unique categories and the percentages sum to consistency with total contig counts
- Every contig in the filtered FASTA has a barcode whose gene assignment is unique (no barcode assigned to a single gene multiple times)
- Total contigs classified (removed + retained) equals the number of contigs in the original cloudrnaSPAdes FASTA

## Limitations

- The filter removes all contigs sharing a barcode flagged as duplicated, so uniquely assigned contigs linked to those barcodes may be collateral losses; the paper reports unique assignments 'remain relatively unchanged', not unchanged exactly
- The ~15% figure is specific to the real SPIso-seq data set and Table 3; simulated human data sets are not the validation target
- Validation depends on the exact barcode-to-gene assignment table for the same data set; mismatched assembly/assignment versions will not reproduce the Table 3 difference
- Article's Discussion section content is absent from the extracted index ('No changelog found'), so no additional discussion-level caveats are available

## Evidence

- [results] Applying the barcode-based filtering between the '1' and '1-fixed' rows of Table 3 removes approximately 15% of the ambiguous assignments, while unique assignments remain relatively unchanged.: [evidence span withheld — anchor status: not_found]
- [abstract] cloudrnaSPAdes assembles isoforms from barcoded RNA-seq linked-read data; validated on simulated and real human data.: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
- [readme] SPAdes manual describes a read-level k-mer coverage filter, distinct from the post-assembly barcode filter reproduced here.: "spades-read-filter is a tool for filtering reads with median kmer coverage less than threshold."
- [readme] SPAdes supports GFA-format assembly graphs, enabling Bandage-based inspection of contigs in the graph.: [evidence span withheld — anchor status: not_found]
