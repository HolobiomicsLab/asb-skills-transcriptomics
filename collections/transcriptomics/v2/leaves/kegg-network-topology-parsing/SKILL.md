---
name: kegg-network-topology-parsing
description: Use when a rule-inference or dynamic-modeling pipeline needs pathway
  topology scaffolds and the source is KEGG (organism code + pathway IDs), or when
  non-KEGG networks must be loaded in GRAPHML format for downstream Boolean rule inference.
when_to_use_negative:
- Your networks come from a non-KEGG source and already exist as GRAPHML files — supply
  them directly via pathwayList instead of downloading KEGG pathways
- You only need pathway over-representation of a DE gene list (conventional ORA),
  not executable Boolean models — the article notes conventional ORA is a different,
  less mechanistic approach
- You have no cell-level condition metadata/contrast file, since the setup and rule
  inference require condition pairs to binarize and compare expression states
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: scBONITA
  role: pipeline.py setup downloads KEGG pathways (--getKEGGPathways, --listOfKEGGPathways,
    --organism) and generates sbatch scripts; pathwayAnalysis.py accepts GRAPHML pathwayList
    files
  repo: https://github.com/Thakar-Lab/scBONITA
- name: KEGG
  role: source of pathway topologies (hsa00010, hsa00020) downloaded for rule inference
- name: SLURM
  role: queue system on which the setup-generated sbatch rule-inference jobs for each
    parsed topology run
- name: GRAPHML
  role: file format for supplying non-KEGG network topologies to scBONITA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:8ccedac45c9270a16048c4efe5bd5281112cbb22c5502c712904d13eff4c5688
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Thakar-Lab/scBONITA
  tools:
  - GRAPHML
  - KEGG
  - Python
  - SLURM
  - scBONITA
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# kegg-network-topology-parsing

## Summary

Retrieve and parse KEGG pathway topologies (e.g., hsa00010, hsa00020 for organism 'hsa') to provide the signed network scaffolds on which scBONITA infers Boolean regulatory rules; non-KEGG networks can be supplied as GRAPHML files.

## When to use

Use when a rule-inference or dynamic-modeling pipeline needs pathway topology scaffolds and the source is KEGG (organism code + pathway IDs), or when non-KEGG networks must be loaded in GRAPHML format for downstream Boolean rule inference.

## When NOT to use

- Your networks come from a non-KEGG source and already exist as GRAPHML files — supply them directly via pathwayList instead of downloading KEGG pathways
- You only need pathway over-representation of a DE gene list (conventional ORA), not executable Boolean models — the article notes conventional ORA is a different, less mechanistic approach
- You have no cell-level condition metadata/contrast file, since the setup and rule inference require condition pairs to binarize and compare expression states

## Inputs

- Organism code and KEGG pathway IDs (e.g., organism 'hsa', pathways '00010', '00020')
- Matrix-style training dataset (tab/comma-delimited; rows = genes/features, columns = cells, expression ~log2(TPM+1), first column labeled 'Genes', first row = cell IDs)
- Metadata/conditions matrix (rows = cells, columns = 0/1 condition labels, first column = CellID matching training data)
- Contrast file listing condition pairs to compare (tab-separated)
- Optional: GRAPHML files for non-KEGG networks (pathwayList argument)

## Outputs

- Downloaded KEGG pathway topology files for each requested pathway ID
- SLURM sbatch scripts for rule inference on each network
- inferred_rules.txt with a sign-compatible canalyzing rule for every node of each KEGG network

## How to apply

Run scBONITA's setup (pipeline.py) with the training matrix, metadata, and contrast file, passing --getKEGGPathways 1, --organism (e.g., hsa), and --listOfKEGGPathways (e.g., '00010' '00020') so the tool downloads the corresponding KEGG pathways for rule inference. Check that pathway size does not exceed --maxNodes (set to the dataset's node count, e.g., 20000). The setup stage generates SLURM sbatch scripts whose genetic-algorithm jobs infer sign-compatible canalyzing Boolean rules per node of each topology. For networks from non-KEGG sources, provide paths to GRAPHML files via the pathwayList argument of pathwayAnalysis.py instead of the KEGG list. Confirm every node of each downloaded network receives an inferred rule in inferred_rules.txt and that all rules are sign-compatible and canalyzing.

## Related tools

- **scBONITA** (pipeline.py setup downloads KEGG pathways (--getKEGGPathways, --listOfKEGGPathways, --organism) and generates sbatch scripts; pathwayAnalysis.py accepts GRAPHML pathwayList files) — https://github.com/Thakar-Lab/scBONITA
- **KEGG** (source of pathway topologies (hsa00010, hsa00020) downloaded for rule inference)
- **SLURM** (queue system on which the setup-generated sbatch rule-inference jobs for each parsed topology run)
- **GRAPHML** (file format for supplying non-KEGG network topologies to scBONITA)

## Examples

```
python3.6 pipeline.py --dataFile "example.csv" --fullPipeline 1 --maxNodes 20000 --separator "," --listOfKEGGPathways "00010" "00020" --getKEGGPathways 1 --organism hsa
```

## Evaluation signals

- Every node of each requested KEGG network (e.g., hsa00010, hsa00020) has an inferred rule present in inferred_rules.txt
- All inferred rules are sign-compatible with the KEGG topology edges and canalyzing
- The setup run reports successful download of the specified KEGG pathways and the number of nodes does not exceed --maxNodes
- SLURM log files show the rule-inference sbatch jobs for each network completed without errors
- Non-KEGG GRAPHML files pass through pathwayAnalysis.py without topology-parsing errors when passed via --pathwayList

## Limitations

- scBONITA is designed for SLURM systems; large datasets (>10G RAM) are not recommended on desktop machines
- Rule inference for large datasets may require adjusting the C sampler's NODE/CELL constants (e.g., 20000 genes x 10000 cells) and recompiling with make
- The C code must be recompiled whenever dataset dimensions exceed the compiled NODE and CELL constants

## Evidence

- [methods] KEGG download for rule inference: "It downloads the KEGG pathways hsa00010 and hsa00020 for rule inference."
- [readme] GRAPHML for non-KEGG networks: "Paths to GRAPHML files that should be used for scBONITA analysis. Usually networks from non-KEGG sources, saved in GRAPHML format."
- [readme] setup invocation example: "The following command runs scBONITA setup for a 20000*10000 comma-separated data set "example.csv". It downloads the KEGG pathways hsa00010 and hsa00020 for rule inference."
- [readme] SLURM job generation: "Step 1 generates sbatch files that enter the specified slurm queue. In a typical use case, these jobs should execute automatically."
