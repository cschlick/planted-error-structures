# planted-error-structures

**40 cryo-EM structures with known, planted coordinate errors**, paired with the deposited
structures they were derived from and a per-structure machine-readable answer key. Built
as the planted-error input class for a study of map-versus-geometry weighting in
real-space refinement; useful as a benchmark for any procedure that claims to *repair*
models — refine the perturbed start against the EMDB map, then score recovery against
the retained truth.

## Contents

```
manifest.csv                       one row per structure
perturbed/<pdb>_perturbed.cif.gz   the deposited model with planted errors (deposited frame)
deposited/<pdb>.cif.gz             the unmodified deposited wwPDB mmCIF (the answer)
answer_keys/<pdb>.json             exactly what was moved, where, and by how much
```

## The perturbation

Per structure: up to **8 segments of 12 consecutive residues**, at sites whose Cα atoms
are pairwise ≥ 18 Å apart (greedy, deterministic site selection), each segment **rigidly
displaced by 2.0 Å** in an independent random direction (`random.Random(seed=1)`, one
unit Gaussian vector per segment, drawn in site order). A couple of structures have 7
segments (a chain too short at one site; recorded in the manifest).

Each `answer_keys/<pdb>.json` records: the planted segments (chain, residue range,
displacement vector in Å, deposited frame), all parameters, and
`interior_atom_i_seq` — the 0-based atom indices (file order) of the **interior**
residues of each segment (the middle 6 of 12; 3 edge residues excluded on each side).
Recovery in the originating study is scored on these interior atoms only, because
segment junctions are pulled by intact neighbours regardless of what the map
contributes.

## Scoring recovery

For a refined model `R`, with perturbed start `P` and deposited truth `T`, over the
interior atoms:

```
recovery = 1 − |R − T| / |P − T|      (mean over interior atoms; |·| = per-atom distance)
```

1.0 = planted error fully repaired; 0.0 = not moved toward truth at all.

## Provenance

- Deposited mmCIFs from a wwPDB rsync mirror synced 2026-07-18/19 (upstream per-file
  revision dates preserved in the source mirror). Maps are referenced by `emdb_id`, not
  redistributed: refinement in the originating study targeted the **half maps** (and
  post-processed copies of half map 1) of the listed EMDB entries.
- Perturbed files regenerated bit-exactly from the deposited models by the study's own
  deterministic code path (verified: identical site selection, moved-atom sets,
  displacement vectors, and interior counts against the stored experiment records).
- The two trial cohorts (`trial_cohort` column): 24 structures used in the
  perturbed-input trials on post-processed maps, 16 in a universality trial
  (8 of them high-resolution).

## Licensing

Deposited structures are wwPDB data (CC0). This dataset, including the perturbed
coordinates and answer keys, is released under CC0.

A companion dataset of raw ModelAngelo builds for the same study:
[modelangelo-builds](https://github.com/cschlick/modelangelo-builds).
