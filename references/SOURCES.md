# Sources

This skill was generated from two primary sources. No long passages were
copied verbatim; the content was read, understood, and condensed into
original guidance. POTCAR files and the manual PDF are not included.

---

## Primary Source: VASP the GUIDE (PDF)

| Field | Value |
|-------|-------|
| Title | VASP the GUIDE |
| Date | August 24, 2005 |
| Authors | Georg Kresse and Juergen Furthmueller |
| Covers | VASP 4.4, 4.5, 4.6 (with VASP 5 preview) |
| Pages | 156 |
| Format | PDF |
| License | Proprietary (VASP Software GmbH) |
| Redistribution | Not permitted without license |

### Pages referenced in the skill

| Skill section | PDF pages | Content |
|---|---|---|
| 1. Input Files | 46-47 | INCAR format, parameter listing |
| 2. POSCAR | 37-40 | POSCAR format and examples |
| 3. KPOINTS | 33-37 | All three KPOINTS formats |
| 4. POTCAR | 33, 131-135 | POTCAR description, element-by-element variants |
| 5. Precision | 49-51 | PREC table, ENCUT, ENAUG |
| 6. Electronic | 69-74 | ALGO, IALGO, RMM-DIIS, mixing parameters |
| 7. Ionic | 55-59 | IBRION, ISIF table, EDIFFG |
| 8. Smearing | 64-66, 119-120 | ISMEAR, SIGMA, metal reference values |
| 9. Spin | 50-51, 88-91 | ISPIN, MAGMOM, non-collinear, SOC |
| 10. Startup | 52-54 | ISTART, ICHARG tables |
| 11. DFT+U | 91-93 | LDAU tags, Dudarev, LMAXMIX |
| 12. Output | 62-63, 76-77 | LORBIT table, LWAVE, LCHARG, LVTOT |
| 13. Real-Space/GGA | 66-69 | LREAL, GGA table |
| 14. Performance | 77-78 | NPAR, LPLANE, NSIM |
| 15. DOS/Bands | 122-124 | DOS and band structure examples |
| 16. MD | 60-62, 128-129 | SMASS, thermostats, BMIX for MD, simulated annealing |
| 17. Recipes | 96-105, 114-115, 120-122 | Bulk relaxation, EOS, Pulay stress, defaults |
| 18. Output Files | Throughout | File descriptions |
| 19. Errors | 71, 74, 114-118 | Convergence, wrap-around, Pulay stress |
| 20. Convergence | 116-120 | Error categories, ENCUT/k-point testing |
| 21. Hybrid/Beyond | 86-88, 93-95, 100-101 | Berry phase, HF/hybrid, HSE, NKRED, ENCUTFOCK |
| 24. Optical | 97-99 | LOPTICS, CSHIFT, LNABLA, LEPSILON, LRPA |
| 25. GW | 99-101 | G0W0 recipe, NOMEGA, LSPECTRAL, ENCUTGW, HSE+GW |
| 27. Surfaces | 129-131 | 5-step surface procedure, FFT grids, dipole correction |
| 28. POTCAR Guide | 131-135 | Element-by-element recommendations, _pv/_sv/_h/_s/_d/_3 |

---

## Secondary Source: VASP Wiki

| Field | Value |
|-------|-------|
| Title | The VASP Manual (online wiki) |
| URL | https://vasp.at/wiki/The_VASP_Manual |
| Covers | VASP 5.x, 6.x (current releases) |
| Accessed | 2026 |
| License | Proprietary (VASP Software GmbH) |

### Wiki pages referenced in the skill

| Skill section | Wiki page(s) | Content |
|---|---|---|
| 4. POTCAR | Available_pseudopotentials | Current element recommendations |
| 14. Performance | Optimizing_the_parallelization | NCORE, KPAR, GPU guidance |
| 21. Hybrid | HFSCREEN | HSE06 screening parameter |
| 22. Meta-GGA | METAGGA | SCAN, r2SCAN, RSCAN, MBJ, LIBXC, deorbitalized |
| 23. Van der Waals | DFT-D3, DFT-D4, Nonlocal_vdW-DF_functionals | D3/D4 tags, vdW-DF INCAR settings |
| 25. GW | GW_calculations | GW workflow, tags |
| 26. Phonons | Phonons_from_finite_differences, Phonons_from_DFPT | IBRION=5/6/7/8, NFREE, LEPSILON |
| 29. ML Force Fields | ML_LMLFF, Machine_learning_force_field_calculations | ML_MODE, train/refit/run workflow |

---

## Version Conflicts

Where the 2005 PDF and the modern wiki disagree on defaults or
behavior, both versions are noted in the skill with explicit version
tags. Key differences:

- PREC: Normal/Accurate not available before VASP 4.5
- ALGO: Fast/VeryFast/Damped/All added in VASP 4.5
- GGA=PE (PBE functional): added in VASP 4.5
- Species names line in POSCAR: optional in VASP 4, required in VASP 5+
- NCORE: VASP 5.2+ only (replaces NPAR)
- KPAR: VASP 5+ only (not in PDF)
- LSORBIT/LNONCOLLINEAR: late beta in VASP 4.5, stable in VASP 5+
- LHFCALC: VASP 5+ only; internal-only documentation at PDF date
- HSE06 (HFSCREEN): not in PDF; available in VASP 5+
- METAGGA: VASP 5.3+ (SCAN from 5.4.4, r2SCAN from 6.2.0)
- IVDW (DFT-D3/D4): D3 in VASP 5+, D4 in VASP 6.6.0+
- LUSE_VDW (vdW-DF): VASP 5.2.12+
- LOPTICS: VASP 5+
- IBRION=6 (symmetry finite diff): VASP 5.1+
- IBRION=7/8 (DFPT): VASP 5.1+
- ML_LMLFF: VASP 6.3+
- vasprun.xml: not documented in PDF; standard in VASP 5+

---

## Licensing Notes

- VASP is proprietary software. A license is required from VASP
  Software GmbH (https://www.vasp.at/) to use the code.
- POTCAR pseudopotential files are part of the VASP distribution and
  cannot be redistributed.
- The PDF manual is distributed with the VASP source code and is not
  freely available.
- This skill contains original, condensed guidance inspired by these
  sources. No long passages were copied.
- The skill itself is released under the MIT license.
