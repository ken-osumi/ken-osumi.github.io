---
title: "Data — I Beat Claude on TNFα for $677"
---


Both arms of both targets, the per-engine scores behind the selection, and the
binding kinetics from Adaptyv Bio. Released under
[ODC-BY](https://opendatacommons.org/licenses/by/1-0/).

| File | What it is |
|---|---|
| [`tnfa_designs.csv`](tnfa_designs.csv) | All 20 TNFα sequences. `Control = TRUE` is the arm with the sequence/structure co-optimization stage removed. |
| [`pdl1_designs.csv`](pdl1_designs.csv) | All 20 PD-L1 sequences, same convention. |
| [`engine_scores.csv`](engine_scores.csv) | Per-design scores from every co-folding engine used for selection: Boltz-2, AlphaFold2, Protenix, RFdiffusion3, OpenFold3, ESMFold2. ipSAE, RMSD, pLDDT, pTM, ipTM, interface PAE. Also the consensus rank each design was selected on. |
| [`tnfa_binding_summary.csv`](tnfa_binding_summary.csv) | Binding calls and fitted kinetics per design: per replicate and overall. `binding`, `binding_strength`, `kd_app` with CIs, `kon`, `koff`, fit quality, the fitted model, and the sequence. |
| [`pdl1_binding_summary.csv`](pdl1_binding_summary.csv) | Same for PD-L1. |
| `tnfa_kinetics/` | The fit-level tables behind the summary: [`fits.csv`](tnfa_kinetics/fits.csv), [`reads.csv`](tnfa_kinetics/reads.csv), [`replicates.csv`](tnfa_kinetics/replicates.csv), [`blanks.csv`](tnfa_kinetics/blanks.csv), plus [the binding models](tnfa_kinetics/binding_models.html) — equations, parameters and units. |

## What is not here

The raw response curves and sensorgram plots — one per concentration per
replicate, 46 MB. Ask and I will send them.

## Caveats that matter when you read these numbers

- **The TNFα run is not closed.** 19 of the 20 designs have both replicates.
  `TNFA_OPT_08` currently has one. Its overall row is that single replicate.
- **`kd_app` is apparent, not monovalent.** TNFα is a homotrimer and the binder
  is the immobilized partner, so avidity is built into this format. Several fits
  sit at the `koff` floor of 1e-5 /s, which means the off-rate was too slow to
  resolve in the dissociation window, not that it was measured to be that value.
- **Antigen:** ACROBiosystems TNA-H4211, the same catalogue number MIT used for
  BoltzGen. Anthropic used TNA-H5228.
- Measured by Adaptyv Bio, BLI, five-point concentration series, duplicate.

Anthropic's data for the same target is at
[Anthropic/claude-protein-binder-design](https://huggingface.co/datasets/Anthropic/claude-protein-binder-design)
under CC-BY-4.0. Every figure in the post can be rebuilt from those two sources.
