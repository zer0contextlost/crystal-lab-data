# Crystal Lab open data

Crystal structures proposed by an AI model, screened with a machine-learned potential, and the most promising
ones re-checked with density functional theory (DFT). Everything is published, including the failures.

AI generators now propose crystals much faster than anyone checks them with physics. This dataset is a small,
open record of what happens when you do check: most candidates that look stable to a machine-learned model turn
out to be known compounds, or less stable than their real competitors, once the competitors are taken seriously.

Built 2026-11-03. Rebuilt nightly while the pipeline runs. Mirrors: [Hugging Face](https://huggingface.co/datasets/zer0contextlost/crystal-lab-data);
raw Quantum ESPRESSO calculations parsed and searchable on [NOMAD](https://nomad-lab.eu/prod/v1/gui/search/entries?upload_id=kNO0ljjeTyK0l_lX9522Ww) (snapshot of 2026-11-01).

## What's in it

| | count |
|---|---|
| Structures generated | 5540 |
| Kept as "discoveries" (stable by ML and not found in reference databases at the time) | 1641 |
| ML hull check finished | 977 |
| ML hull check at or below 0 eV/atom | 0 |
| DFT jobs (passes / fails / still running or queued) | 9 (2 / 2 / 5) |

### How the prompts were chosen, and how each way does
On 2026-11-01 the autopilot switched from random formulas to mostly substitution-based prompts (see METHODS.md).
This table compares the sources; "within 0.05" counts ML hull results at most 0.05 eV/atom above the hull.
New rows need a few nights of hull checks before their numbers mean much.

| prompt mode | structures | new | ML hull checked | median e_hull (eV/atom) | within 0.05 | DFT passes |
|---|---|---|---|---|---|---|
| random formula (before 11-01) | 3713 | 1327 | 876 | 0.358 | 1 | 0 |
| surprise (before 11-01) | 1681 | 266 | 96 | 0.136 | 14 | 2 |
| substitution | 67 | 32 | 3 | 0.178 | 0 | 0 |
| surprise | 52 | 7 | 0 | - | 0 | 0 |
| random | 17 | 9 | 2 | 0.319 | 0 | 0 |
| human | 10 | 0 | 0 | - | 0 | 0 |

ML verdicts: unstable 2016, rock solid 1365, plausible 1249, borderline 742, falls apart 168.
Novelty at generation time: skipped 2926, new composition 1004, known 968, new structure 637, rediscovered 5.

### Files
- `structures.jsonl.gz`: one JSON object per structure: formula, space group, CIF, how the prompt was chosen
  (`prompt_mode`, from 2026-11-01), ML energies and verdict,
  novelty check (with the matching database IDs when it was already known), ML hull result, DFT summary.
- `structures.csv`: the same without CIFs, for spreadsheets.
- `dft_results.csv`: one row per DFT job.
- `dft/<id>/`: every DFT job: `candidate.cif`, competitor structures (`comp/`), Quantum ESPRESSO inputs and
  outputs for the candidate and each competitor (`runs/`), `result.json`, and for passes the properties
  (`props/`: band gap, DOS, elastic tensor) and phonon runs (`phonon/`). `.out` files are gzipped.
- `stats.json`: the counts above. `build.log`: files left out of this build and why.
- `METHODS.md`: how every number was produced, and what it does and doesn't mean.

## How to read the results
- An ML "rock solid" verdict means a low formation energy with MACE. It says nothing about competing phases.
- The ML hull compares against Alexandria near-hull phases computed with the same ML model. `pending` rows carry
  only `previous_e_hull_eV_atom`, a superseded value from before a stricter re-check; don't use it as a result.
- A DFT **pass** means: with PBE, the candidate is at most 0.025 eV/atom above the convex hull built from its
  Alexandria competitors. It does **not** mean the material has been made, or that it can be made. PBE errors
  of a few tens of meV/atom are normal, and the hull only contains the competitors we checked.
- Phonons are a screen on a small supercell (shortest side ~7 Å) for imaginary modes, not a full study.

## Licence and credit
Our data (generated structures, ML and DFT results): **CC BY 4.0**, credit "Crystal Lab open data, zer0contextlost".
Competitor structures come from the **Alexandria** database (Schmidt et al., CC BY 4.0); Alexandria and
Materials Project IDs are given wherever a structure matched them. Pseudopotentials are SSSP 1.1.2 efficiency
(Prandini et al.) and are not redistributed here. See `ATTRIBUTION.md`.

Structures from licensed databases (e.g. CCDC/ICSD) used in any check are never included; such checks are cited
by deposition number only.

Corrections and questions: open an issue.
