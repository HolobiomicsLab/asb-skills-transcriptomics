---
name: pancreas-endocrine-signaling-regulatory-analysis
description: Use when analyzing TFvelo output on the pancreas dataset to check whether
  high-|weight| transcription factors (REST, HMGN3) act with consistent signs on insulin
  secretion pathway genes and whether KEGG endocrine terms are enriched among -fitted
  genes.
when_to_use_negative:
- The TFvelo run for the pancreas dataset has not been executed yet ('TFvelo_pancreas/rc.h5ad'
  does not exist)
- The dataset lacks transcription factor modeling output, so no TF weight matrix is
  available for sign consistency checks
- The goal is splicing-based velocity estimation (unspliced/spliced counts); scVelo's
  dynamical model, not TFvelo's TF-weight analysis, is the appropriate tool
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: TFvelo
  role: Produces the pancreas run results (rc.h5ad) with best-fitted genes and per-TF
    weight matrix analyzed in this skill
  repo: github.com/xiaoyeye/TFvelo
- name: gseapy
  role: Runs over-representation enrichment (Fisher exact test) on the best-fitted
    gene list against the KEGG library
- name: scVelo
  role: Framework upon which TFvelo is built; provides the underlying code base for
    the pancreas run
  repo: theislab/scvelo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:7ed9f688d0730e32b0f9cf7c318be468398304e59e2f25c9f8f76af99b809111
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TFvelo
  - gseapy
  - scVelo
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pancreas-endocrine-signaling-regulatory-analysis

## Summary

Reproduces the TFvelo pancreas analysis that links KEGG insulin-secretion enrichment of best-fitted genes to TF weight signatures, verifying REST as a consistently negative-weight and HMGN3 as a consistently positive-weight regulator on insulin secretion pathway genes.

## When to use

Use when analyzing TFvelo output on the pancreas dataset to check whether high-|weight| transcription factors (REST, HMGN3) act with consistent signs on insulin secretion pathway genes and whether KEGG endocrine terms are enriched among best-fitted genes.

## When NOT to use

- The TFvelo run for the pancreas dataset has not been executed yet ('TFvelo_pancreas/rc.h5ad' does not exist)
- The dataset lacks transcription factor modeling output, so no TF weight matrix is available for sign consistency checks
- The goal is splicing-based velocity estimation (unspliced/spliced counts); scVelo's dynamical model, not TFvelo's TF-weight analysis, is the appropriate tool

## Inputs

- TFvelo results file 'TFvelo_pancreas/rc.h5ad' containing best-fitted genes and TF weight matrix
- KEGG gene-set library (via gseapy.enrichr)
- Insulin secretion pathway gene list

## Outputs

- KEGG enrichment table (Fisher exact test results) saved as 'KEGG_enrichment_results.csv'
- REST/HMGN3 weight-sign verification on insulin secretion pathway genes
- Count of TFs with absolute normalized weight > 0.5

## How to apply

Load the TFvelo run results from 'TFvelo_pancreas/rc.h5ad' produced by 'python TFvelo_run_demo.py --dataset_name pancreas' and extract the best-fitted genes and their TF weight matrix. Run over-representation enrichment on the best-fitted gene list with gseapy.enrichr against the KEGG library, which applies a Fisher exact test, and confirm that insulin secretion and glucagon signaling KEGG terms are significantly enriched (Fig. 3b). Restrict the TF weight matrix to genes belonging to the insulin secretion pathway, normalize the TF weights, and identify TFs whose absolute normalized weight exceeds 0.5. Verify that REST exhibits consistently negative weights and HMGN3 consistently positive weights on insulin secretion pathway genes, with both TFs among the high-absolute-weight regulators. Write the enrichment table together with the REST/HMGN3 weight verification results to 'KEGG_enrichment_results.csv'.

## Related tools

- **TFvelo** (Produces the pancreas run results (rc.h5ad) with best-fitted genes and per-TF weight matrix analyzed in this skill) — github.com/xiaoyeye/TFvelo
- **gseapy** (Runs over-representation enrichment (Fisher exact test) on the best-fitted gene list against the KEGG library)
- **scVelo** (Framework upon which TFvelo is built; provides the underlying code base for the pancreas run) — theislab/scvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas
```

## Evaluation signals

- Insulin secretion and glucagon signaling KEGG terms appear as enriched terms in the Fisher-exact enrichment table (as in Fig. 3b)
- REST exceeds the normalized |weight| > 0.5 threshold with consistently negative weights on insulin secretion pathway genes
- HMGN3 exceeds the normalized |weight| > 0.5 threshold with consistently positive weights on insulin secretion pathway genes
- Output file 'KEGG_enrichment_results.csv' contains both the enrichment table and REST/HMGN3 weight verification results
- The 'TFvelo_pancreas/rc.h5ad' file was regenerated with default parameters matching the manuscript run

## Limitations

- TFvelo is slower than scVelo in the same framework, which could be a weakness when applied to large-scale datasets
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14), so some genes may be absent from the best-fitted gene list used for enrichment
- The current linear model for estimating TF weights may be insufficient; more complex models may be needed, which could affect weight sign consistency

## Evidence

- [results] When analyzing TFvelo's weights on genes within the insulin secretion pathway, REST consistently has a negative weight, while HMGN3 consistently exhibits a positive weight.: [evidence span withheld — anchor status: not_found]
- [methods] Run over-representation enrichment on the best-fitted gene list with gseapy.enrichr using the KEGG gene-set library, which applies a Fisher exact test.: [evidence span withheld — anchor status: not_found]
- [methods] count TFs whose absolute normalized weight exceeds 0.5: [evidence span withheld — anchor status: not_found]
- [readme] This will automatically download, preprocess and run TF model on pancrease dataset. The result will be stored in 'TFvelo_pancreas/rc.h5ad'.: "This will automatically download, preprocess and run TF model on pancrease dataset. The result will be stored in 'TFvelo_pancreas/rc.h5ad'."
- [discussion] the current linear model in TFvelo for estimating the weight of each TFs: "more complex models may be explored to replace the current linear model in TFvelo for estimating the weight of each TFs"
