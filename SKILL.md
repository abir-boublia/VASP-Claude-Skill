---
name: "vasp"
description: "When the user asks for help setting up, running, debugging, or interpreting VASP density functional theory calculations, including INCAR parameters, POSCAR/KPOINTS/POTCAR files, convergence, relaxation, electronic structure, DFT+U, hybrid functionals, meta-GGA, van der Waals corrections, phonons, optical properties, GW, machine-learned force fields, or common errors."
---

# VASP Expert Skill

You are an expert in VASP (Vienna Ab initio Simulation Package), the plane-wave DFT code developed by Georg Kresse and Juergen Furthmueller at the University of Vienna. VASP is proprietary software requiring a license from the VASP Software GmbH. This skill draws on "VASP the GUIDE" (PDF, August 2005, covering VASP 4.4-4.6 with VASP 5 preview) and the official VASP wiki at https://vasp.at/wiki/The_VASP_Manual for modern VASP 5/6 features.

## Core Principles

- Always produce syntactically valid INCAR, POSCAR, and KPOINTS files.
- State the VASP version when a feature or default changed between versions. Flag version conflicts rather than guessing.
- Recommend PAW potentials (the modern default) over ultrasoft pseudopotentials unless the user specifies otherwise.
- Keep INCAR files minimal — only set tags that differ from reasonable defaults.
- Never include POTCAR file contents; POTCAR files are proprietary and cannot be redistributed.
- Cite the VASP wiki section or PDF page when introducing non-trivial parameters.
- Prefer well-tested, conservative settings. When in doubt, start with tighter convergence.

---

## 1. Input Files Overview

Every VASP calculation requires four input files in the working directory:

| File | Purpose |
|------|---------|
| **INCAR** | Calculation parameters (tagged format: `TAG = value`) |
| **POSCAR** | Crystal structure (lattice vectors + atomic positions) |
| **KPOINTS** | Brillouin zone sampling (k-point mesh) |
| **POTCAR** | Pseudopotentials (concatenated, one block per species) |

VASP reads these four files and produces output files (OUTCAR, OSZICAR, CONTCAR, CHGCAR, WAVECAR, DOSCAR, EIGENVAL, etc.). The order of species in POTCAR must match the order in POSCAR.

### INCAR syntax
- One tag per line: `TAG = value`
- Comments: `#` or `!` after a value
- Continuation: `\` at end of line
- Tags are case-insensitive; values are type-specific (integer, real, logical, string)

---

## 2. POSCAR Format

The POSCAR file defines the crystal structure:

```
FCC Silicon                    # Comment line (system name)
  5.43                         # Universal scaling factor (Angstrom)
  0.5  0.5  0.0               # Lattice vector a1
  0.0  0.5  0.5               # Lattice vector a2
  0.5  0.0  0.5               # Lattice vector a3
Si                             # Species names (VASP 5+; optional in VASP 4)
  2                            # Number of atoms per species
Selective dynamics             # Optional: enable per-atom constraints
Direct                         # Direct (fractional) or Cartesian coordinates
  0.00  0.00  0.00  T T T     # Position + selective dynamics flags
  0.25  0.25  0.25  T T T
```

### Key rules
- The scaling factor multiplies all lattice vectors. A negative value is interpreted as the desired cell volume in Angstrom^3 (VASP scales vectors to match).
- Species names line is required in VASP 5+ but absent in VASP 4.x. Include it for portability.
- Selective dynamics flags (T/F per direction) appear only when the `Selective dynamics` line is present.
- For molecular dynamics, an optional block of velocities follows the positions (Angstrom/fs in Cartesian, or lattice units in Direct).
- CONTCAR has the same format as POSCAR and serves as the restart geometry.

---

## 3. KPOINTS Formats

Three main formats for the KPOINTS file:

### 3.1 Automatic mesh (most common)

```
Automatic mesh               # Comment
0                            # 0 = automatic generation
Gamma                        # Gamma-centered (or Monkhorst for Monkhorst-Pack)
  8  8  8                    # Subdivisions along b1, b2, b3
  0  0  0                    # Optional shift
```

Use `Gamma` for hexagonal cells and whenever the Gamma-point must be included. Use `Monkhorst` (Monkhorst-Pack) for cubic/orthorhombic cells. For hexagonal lattices, Gamma-centered grids preserve symmetry; Monkhorst-Pack grids may break it.

### 3.2 Explicit k-points (atoms, molecules, or special grids)

```
Explicit k-points
4                            # Number of k-points
Reciprocal                   # Reciprocal or Cartesian
  0.0  0.0  0.0   1.0       # k-point coordinates + weight
  0.5  0.0  0.0   2.0
  0.0  0.5  0.0   2.0
  0.5  0.5  0.0   1.0
```

### 3.3 Line-mode (band structures)

```
Line-mode                    # Comment
40                           # Points per segment
Line-mode                    # Must repeat "Line-mode" or "L"
Reciprocal                   # Reciprocal coordinates
  0.0  0.0  0.0    ! Gamma
  0.5  0.0  0.5    ! X

  0.5  0.0  0.5    ! X
  0.5  0.25 0.75   ! W
```

Pairs of lines define path segments. A blank line between segments separates disconnected paths. Used with ICHARG=11 for non-self-consistent band structure calculations.

---

## 4. POTCAR Guidelines

- POTCAR files are proprietary; never redistribute them. Concatenate from the VASP potential library.
- Order must match species order in POSCAR.
- Use PAW potentials (recommended for VASP 5+): `potpaw_PBE` or `potpaw_LDA`.
- Choose the appropriate variant: standard, `_sv` (semi-core), `_pv` (p-valence), `_d` (with d), `_h` (hard), `_s` (soft).
- Check ENMAX in the POTCAR header — it determines the default ENCUT.
- For multi-species calculations, VASP uses `max(ENMAX)` from all POTCAR blocks for PREC=Normal/Accurate, or `max(ENMAX)*1.3` for PREC=High.

```bash
# Typical POTCAR construction (do not redistribute the result)
cat /path/to/potpaw_PBE/Si/POTCAR /path/to/potpaw_PBE/O/POTCAR > POTCAR
```

### Variant selection by element group (PDF pp. 131-135)

| Group | Recommended variant | Notes |
|-------|-------------------|-------|
| Li, Be | `Li_sv`, `Be_sv` | 1s in valence for accuracy |
| Na, K, Rb, Cs | `Na_pv`, `K_sv`, `Rb_sv`, `Cs_sv` | Semi-core p or s states essential |
| Mg, Ca, Sr, Ba | `Mg`, `Ca_pv`, `Sr_sv`, `Ba_sv` | Ca needs p-valence; Sr/Ba need s-valence |
| 3d metals (Sc-Zn) | Standard or `_pv`/`_sv` | Use `_pv` for early 3d (Sc-V) where semi-core matters |
| 4d/5d metals | Standard or `_pv` | Check d(E) correction tables in PDF for US-PP accuracy |
| O, S, Se, Te | Standard | `O_s` (soft) for quick tests; `O_h` (hard) for high accuracy |
| N, P, As | Standard | `N_s` for large cells; `N_h` for precise forces |
| Rare earths (Ce-Lu) | Standard or `_3` | `_3` treats f-electrons as core — required for non-LDA+U |

- **Rule**: Use `_pv`/`_sv` variants when semi-core states participate in bonding or hybridize with valence bands. For alkali/alkaline-earth metals, this is almost always the case.
- See the VASP wiki for current element-by-element recommendations: https://vasp.at/wiki/Available_pseudopotentials

---

## 5. INCAR: Precision and Energy Cutoff

### PREC — Precision

| Value | ENCUT default | NGx default | FFT grid | Notes |
|-------|--------------|-------------|----------|-------|
| Low | smallest ENMAX | ~3/4 of exact | Coarse | Quick tests only |
| Medium | largest ENMAX | ~3/4 of exact | Medium | Default prior to VASP 4.5 |
| **Normal** | largest ENMAX | exact | exact | **Recommended for everyday** |
| **Accurate** | largest ENMAX | exact | exact | **Recommended when forces matter** |
| High | largest ENMAX * 1.3 | exact | exact | Deprecated; use Accurate + explicit ENCUT |

**Version note**: Normal and Accurate are available from VASP 4.5 onward. Earlier versions only have Low/Medium/High. (PDF p. 50)

### ENCUT — Plane-wave energy cutoff (eV)

- Default: read from POTCAR (ENMAX). Override explicitly for convergence tests.
- For comparing energies of different compositions, fix ENCUT to the maximum ENMAX across all species.
- Typical bulk calculations: 50-100 plane waves per atom. A larger basis set is rarely needed and may indicate a bad pseudopotential.
- For volume relaxation, increase ENCUT by 1.3x the default to reduce Pulay stress errors. (PDF p. 114-115)

### ENAUG — Augmentation charge cutoff (eV)

- Only relevant for PAW. Default from POTCAR (EAUG).
- Set explicitly when comparing different compositions to ensure identical augmentation grids.

---

## 6. INCAR: Electronic Minimization

### ALGO — Algorithm

| Value | IALGO | Method | Best for |
|-------|-------|--------|----------|
| Normal | 38 | Blocked Davidson | Default; robust |
| VeryFast | 48 | RMM-DIIS | Large systems; needs good starting WAVECAR |
| Fast | 38+48 | Davidson then RMM-DIIS | **General recommendation** |
| All | 58 | Conjugate gradient | Exact diagonalization; meta-GGA |
| Damped | 53 | Damped velocity Verlet | HF/hybrid calculations (required) |

**Version note**: Algo=Fast, VeryFast, Damped, All are VASP 4.5+. Earlier versions use IALGO directly. (PDF pp. 69-71)

**Caution**: RMM-DIIS (IALGO=48/VeryFast) can be unstable if the starting wavefunctions are poor. It may converge to a wrong solution for metallic systems. If total energy oscillates, switch to ALGO=Normal or ALGO=All. (PDF p. 71)

### EDIFF — Electronic convergence criterion (eV)

- Default: 1E-4 eV. Tighten to 1E-6 for forces or phonon calculations.
- VASP stops the electronic self-consistency loop when the total energy change between steps is smaller than EDIFF.

### NELM, NELMIN, NELMDL — Iteration limits

- NELM: Maximum electronic steps (default 60). Increase for difficult convergence.
- NELMIN: Minimum electronic steps (default 2). Set to 4-6 for MD to ensure converged forces.
- NELMDL: Initial non-self-consistent steps (default depends on ISTART). Negative values use the Harris-Foulkes approximation for initial delay steps.

### Mixing parameters for difficult convergence

When the default mixer fails (magnetic systems, slabs, surfaces), try: (PDF p. 74)

```
AMIX     = 0.2
BMIX     = 0.0001
AMIX_MAG = 0.8
BMIX_MAG = 0.0001
```

For MD with ALGO=VeryFast, use `BMIX = 2.0` and `MAXMIX = 50` to stabilize charge sloshing between steps. (PDF p. 128)

---

## 7. INCAR: Ionic Relaxation

### IBRION — Relaxation algorithm

| Value | Method | When to use |
|-------|--------|-------------|
| -1 | No update (static) | Single-point energy, DOS |
| 0 | Molecular dynamics (Verlet) | Ab initio MD |
| 1 | Quasi-Newton (RMM-DIIS) | Near equilibrium; fastest near minimum |
| **2** | **Conjugate gradient** | **Safe default for most relaxations** |
| 3 | Damped MD | Difficult relaxations; good for surfaces |
| 5 | Finite differences (all atoms) | Phonons, vibrational analysis (VASP 4.5+) |
| 6 | Finite differences (symmetry-reduced) | Phonons using symmetry (VASP 5.1+) |
| 7 | DFPT perturbation (all atoms) | DFPT phonons (VASP 5.1+) |
| 8 | DFPT perturbation (symmetry-reduced) | DFPT phonons using symmetry (VASP 5.1+) |

See Section 26 for phonon calculation details.

### NSW — Number of ionic steps

- Default: 0 (static calculation). Set to 100-500 for relaxations; VASP stops early if EDIFFG is satisfied.

### ISIF — Stress and relaxation control

| ISIF | Forces | Stress | Relax ions | Relax shape | Relax volume |
|------|--------|--------|------------|-------------|--------------|
| 0 | Yes | No | Yes | No | No |
| **2** | **Yes** | **Yes** | **Yes** | **No** | **No** |
| **3** | **Yes** | **Yes** | **Yes** | **Yes** | **Yes** |
| 4 | Yes | Yes | Yes | Yes | No |
| 5 | Yes | Yes | No | Yes | No |
| 6 | Yes | Yes | No | Yes | Yes |
| 7 | Yes | Yes | No | No | Yes |

- **ISIF=2**: Ion-only relaxation (cell fixed). The safe default.
- **ISIF=3**: Full relaxation (ions + cell shape + volume). Use ENCUT >= 1.3 * ENMAX to reduce Pulay stress.
- **ISIF=4**: Relax ions and cell shape at constant volume. Preferred method for accurate bulk parameters: relax at several fixed volumes and fit to an equation of state. (PDF p. 115)
- **ISIF >= 3 with IBRION=6**: Also computes elastic constants (clamped-ion and relaxed-ion). See VASP wiki.

### EDIFFG — Ionic convergence criterion

- Default: EDIFF * 10. Negative value = force criterion (eV/Angstrom).
- **Recommended**: Use a negative value, e.g., `EDIFFG = -0.02` (forces converged to 0.02 eV/Angstrom). Force criteria are more intuitive and reliable. (PDF p. 55)

### POTIM — Step size

- For relaxations (IBRION=1,2): scaling factor for the trial step. Default 0.5. Reduce to 0.1-0.2 for sensitive systems.
- For MD (IBRION=0): time step in femtoseconds. Must be set explicitly (no reliable default). Typical: 1-2 fs.
- For finite differences (IBRION=5,6): displacement in Angstrom. If too large, VASP 5.1+ defaults to 0.015 Angstrom.

---

## 8. INCAR: Smearing and k-point Integration

### ISMEAR — Smearing method

| Value | Method | Use case |
|-------|--------|----------|
| **-5** | **Tetrahedron + Bloechl corrections** | **Best for semiconductors/insulators: total energy and DOS** |
| -1 | Fermi smearing | Finite-temperature DFT |
| **0** | **Gaussian smearing** | **Relaxations; safe for all systems** |
| 1..N | Methfessel-Paxton order N | **Metals: accurate forces and total energy** |

### SIGMA — Smearing width (eV)

- Default: 0.2 eV. For metals with Methfessel-Paxton, choose SIGMA so that `entropy T*S` in OUTCAR is < 1 meV/atom.
- For ISMEAR=-5 (tetrahedron), SIGMA is ignored.

### Reference SIGMA values for metals (PDF p. 119)

| Metal | Recommended SIGMA (eV) |
|-------|----------------------|
| Al | 1.0 |
| Li | 0.4 |
| Te | 0.8 |
| Cu, Pd | 0.4 |
| V, Rh | 0.2 |
| K | 0.3 |

These values keep the entropy contribution below ~1 meV/atom with Methfessel-Paxton order 1. Always verify by checking `entropy T*S` in OUTCAR.

### Recommendations (PDF pp. 64-66, 119)
- **Semiconductors/insulators**: ISMEAR=-5 for final energy/DOS. ISMEAR=0 for relaxations if the cell is too large for tetrahedron method.
- **Metals**: ISMEAR=1 or 2 (Methfessel-Paxton) for relaxations. ISMEAR=-5 for accurate total energy (no relaxation). Never use ISMEAR>0 for semiconductors.
- **DOS calculations**: Always ISMEAR=-5 (tetrahedron). This gives the most accurate DOS for any system.
- Always check `entropy T*S` in OUTCAR. If it exceeds a few meV/atom, reduce SIGMA.

---

## 9. INCAR: Spin Polarization

### ISPIN — Spin switch

- `ISPIN = 1`: Non-spin-polarized (default). Faster; use when magnetism is absent.
- `ISPIN = 2`: Spin-polarized. Required for magnetic systems (Fe, Ni, Co, Mn oxides, etc.).

### MAGMOM — Initial magnetic moments

- Default: NIONS * 1.0 (all atoms start with moment 1.0 muB).
- Set explicitly for magnetic systems to guide convergence:

```
MAGMOM = 2*5.0 4*0.6    # 2 Fe atoms at 5 muB, 4 O atoms at 0.6 muB
```

- For antiferromagnetic ordering:

```
MAGMOM = 5.0 -5.0 0.6 0.6 0.6 0.6   # AF: Fe up, Fe down, 4 O
```

- **Convergence tip**: For difficult magnetic systems, first converge non-magnetically (ISPIN=1), then restart with ISPIN=2, ICHARG=1 to read the converged charge density. (PDF p. 51)

### Non-collinear magnetism (VASP 4.5+)
- `LNONCOLLINEAR = .TRUE.`: Enables non-collinear spin calculations.
- MAGMOM requires three values per atom (x, y, z components):

```
MAGMOM = 1 0 0   0 1 0     # Atom 1: moment along x, Atom 2: along y
```

- Recommended workflow: converge collinear first, then restart with LNONCOLLINEAR. (PDF p. 88)

### Spin-orbit coupling (VASP 4.5+)
- `LSORBIT = .TRUE.`: Enables spin-orbit coupling (automatically sets LNONCOLLINEAR=.TRUE.).
- Requires PAW potentials. Not supported with ultrasoft pseudopotentials.
- `SAXIS = sx sy sz`: Quantization axis for spin. Default: (0+, 0, 1) (approximately x-direction).
- Set `NBANDS = 2 * (number of bands from collinear run)` for SOC calculations.
- **Warning**: Spin-orbit coupling was in late beta in VASP 4.6; modern VASP 5/6 has stable SOC support. (PDF pp. 88-90)

---

## 10. INCAR: Startup and Restart

### ISTART — Wavefunction initialization

| Value | Meaning | When to use |
|-------|---------|-------------|
| **0** | **Start from scratch** | **First calculation; no WAVECAR** |
| 1 | Continue, constant energy cutoff | Restart from prior WAVECAR (same ENCUT) |
| 2 | Continue, constant basis set | Restart when ENCUT changed |
| 3 | Full restart with charge extrapolation | MD restarts |

### ICHARG — Charge density initialization

| Value | Meaning | When to use |
|-------|---------|-------------|
| 0 | From wavefunctions (ISTART>0) | Normal continuation |
| 1 | Read CHGCAR from prior run | Restart with new wavefunction init |
| **2** | **Atomic superposition** | **Starting from scratch (default when ISTART=0)** |
| **11** | **Non-self-consistent from CHGCAR** | **Band structures and DOS from converged charge** |
| 12 | Non-self-consistent Harris-Foulkes | Rare; testing only |

**Critical**: For band structure calculations, always use ICHARG=11 with a converged CHGCAR from a prior self-consistent run. (PDF p. 53)

---

## 11. INCAR: DFT+U

The L(S)DA+U method corrects the self-interaction error for localized d and f electrons. (VASP 4.6+; PDF pp. 91-93)

### Key tags

```
LDAU     = .TRUE.          # Enable DFT+U
LDAUTYPE = 2               # 1=Liechtenstein, 2=Dudarev (default), 4=LDA+U
LDAUL    = 2  -1           # l-quantum: -1=off, 1=p, 2=d, 3=f (per species)
LDAUU    = 4.0  0.0        # U parameter (eV, per species)
LDAUJ    = 0.0  0.0        # J parameter (eV, per species)
LDAUPRINT = 1              # 0=silent, 1=occupancy in OUTCAR, 2=verbose
```

### Important notes
- LDAUL, LDAUU, and LDAUJ must be specified for **all** atomic species, even those without +U correction (set LDAUL=-1 for those).
- For Dudarev's approach (LDAUTYPE=2), only (U-J) matters; U and J are not independent.
- Total energies from calculations with different U/J values are not comparable.
- For band structures with DFT+U, increase `LMAXMIX` to 4 (d-elements) or 6 (f-elements) to ensure correct CHGCAR. (PDF p. 93)
- DFT+U calculations may need increased `LMAXMIX` for convergence of the charge density mixer.

### Example: NiO with DFT+U

```
# INCAR for NiO DFT+U
ISPIN   = 2
MAGMOM  = 2*2.0 2*0.0    # AF Ni, non-magnetic O
LDAU    = .TRUE.
LDAUTYPE = 2
LDAUL   = 2 -1           # d-correction on Ni, none on O
LDAUU   = 6.4 0.0
LDAUJ   = 0.0 0.0
LMAXMIX = 4
```

---

## 12. INCAR: Output Control

### LWAVE — Write WAVECAR

- Default: .TRUE. Set `.FALSE.` to save disk for production runs where restart is not needed.

### LCHARG — Write CHGCAR/CHG

- Default: .TRUE. Keep `.TRUE.` if you plan DOS or band structure follow-up runs (ICHARG=11 needs CHGCAR).

### LORBIT — Projected DOS and PROCAR

| Value | Output | Requires RWIGS? |
|-------|--------|-----------------|
| 0 | DOSCAR + PROCAR (spd) | Yes |
| 1 | DOSCAR + PROCAR (spd) | Yes |
| 2 | DOSCAR + PROCAR (spd, phase) | Yes |
| 5 | DOSCAR + PROCAR (spd) | Yes |
| **10** | **DOSCAR + PROCAR (spd)** | **No (PAW)** |
| **11** | **DOSCAR + PROCAR (lm-decomposed)** | **No (PAW)** |
| 12 | DOSCAR + PROCAR (lm, phase) | No (PAW) |

- **Recommended**: LORBIT=11 with PAW potentials (no RWIGS needed, gives full lm-decomposed PROCAR). (PDF pp. 62-63)

### LVTOT — Write LOCPOT

- Default: .FALSE. Set to .TRUE. for work function calculations or electrostatic potential analysis.
- Core-level shifts can be extracted from LOCPOT by comparing the average electrostatic potential at ionic sites. (VASP 4.4.4+; PDF p. 77)

### NEDOS, EMIN, EMAX — DOS energy grid

- NEDOS: Number of grid points for DOSCAR (default 301). Increase to 2001-3001 for fine DOS.
- EMIN/EMAX: Energy range for DOS (default: from eigenvalue range).

---

## 13. INCAR: Real-Space Projection and GGA

### LREAL — Projection operator

| Value | Method | When to use |
|-------|--------|-------------|
| .FALSE. | Reciprocal space (exact) | Default; small cells (<20 atoms) |
| Auto | Real-space with automatic optimization | **Recommended for >20 atoms with PAW** |
| On / .TRUE. | Real-space (old projectors) | Legacy; Auto preferred |

- Switching between LREAL=.FALSE. and LREAL=Auto introduces a small, systematic error. Keep LREAL consistent across related calculations.
- ROPT controls the accuracy of real-space projectors. Default depends on PREC. (PDF pp. 66-68)

### GGA — Exchange-correlation functional

| Value | Functional | Notes |
|-------|-----------|-------|
| PW | Perdew-Wang 91 | |
| PB | Perdew-Burke-Ernzerhof (original) | Older tag |
| PE | PBE | **VASP 4.5+; recommended** |
| RP | Revised PBE (RPBE) | Better surface adsorption energies |
| PS | PBEsol | Improved lattice constants for solids |
| 91 | Perdew-Wang 91 | Same as PW |
| LM | Langreth-Mehl-Hu | |

- Leave GGA unset to use LDA (or the functional matching the POTCAR).
- PBE (GGA=PE) is the most widely used GGA for solids. Use with `potpaw_PBE` POTCARs.
- PBEsol (GGA=PS) often gives better lattice constants than PBE for solids.
- **Do not mix** LDA POTCARs with GGA functionals or vice versa.

---

## 14. INCAR: Performance and Parallelization

### NCORE — Cores per orbital band (VASP 5.2+)

- Controls how many cores work on each orbital. NCORE replaces NPAR (NPAR = total_cores / NCORE).
- **Recommended**: Set NCORE to the number of cores per NUMA domain on the node. For example, NCORE=8 on a 32-core AMD Epyc node with 4 NUMA domains.
- Cannot be set when using OpenMP threading or GPU ports — the thread count controls parallelization instead.
- See: https://vasp.at/wiki/Optimizing_the_parallelization

### NPAR — Band parallelization

- Tells VASP to parallelize over bands. NPAR = number of bands treated in parallel.
- Default: 1 (all cores work on plane-wave coefficients for one band at a time).
- **Rule of thumb**: NPAR = sqrt(number of cores) for most clusters. (PDF p. 77)
- NPAR = number of nodes for slow-network clusters.
- In modern VASP, prefer setting NCORE instead of NPAR.

### KPAR — k-point parallelization (VASP 5+)

- Divides k-points across groups of MPI ranks. Total ranks must be divisible by KPAR.
- Very efficient for systems with many k-points; try KPAR up to the number of irreducible k-points if memory allows.
- Choose KPAR that evenly divides the k-point count to minimize communication overhead.
- For small bulk cells with many k-points, NCORE=1 and KPAR=NKPTS can be optimal.
- On GPUs: set KPAR = number of GPUs if memory allows.
- **Tip**: If the irreducible k-point count is prime, copy IBZKPT to KPOINTS and add zero-weighted k-points to improve factorization.
- **Version note**: KPAR is a VASP 5+ feature not in the 2005 PDF.

### LPLANE — Plane-wise data distribution

- `.TRUE.`: Distributes data plane-by-plane in real space. Reduces communication bandwidth for FFTs.
- Use LPLANE=.TRUE. when NGZ >= 3*(cores)/NPAR. (PDF p. 77)

### NSIM — Blocked band optimization

- Number of bands optimized simultaneously in RMM-DIIS. Default: 4.
- Increasing NSIM (e.g., 4-8) can improve cache efficiency on modern hardware.
- On GPUs, increase NSIM up to 32 for better performance.

### General performance tips

- Run a dry run with `ALGO=None` (or `--dry-run` in VASP 6) to check NBANDS and plane-wave counts without running an expensive calculation.
- Benchmark with a few electronic or ionic steps before committing to a long production run. Compare the LOOP timer per step in OUTCAR.
- Set `LWAVE = .FALSE.` and `LCHARG = .FALSE.` for production MD runs to avoid writing large files every ionic step.
- Set `LREAL = Auto` for cells with >20 atoms (2-5x speedup for >100 atoms).
- Bind MPI ranks to cores: set `OMP_PROC_BIND=true` and `OMP_PLACES=cores`.

---

## 15. DOS and Band Structure Calculations

### Step 1: Self-consistent ground state

```
# INCAR for SCF calculation
ISTART  = 0
ICHARG  = 2
ISMEAR  = -5        # Tetrahedron for semiconductors; 1 for metals
SIGMA   = 0.2
LORBIT  = 11         # Projected DOS
NEDOS   = 2001       # Fine DOS grid
LCHARG  = .TRUE.     # Must write CHGCAR
LWAVE   = .TRUE.     # Useful for restarts
```

Use a converged k-point mesh (e.g., 11x11x11 for bulk). Run the calculation to full convergence.

### Step 2: Non-SCF DOS (optional, for finer mesh)

```
# INCAR for non-SCF DOS
ICHARG  = 11         # Read CHGCAR, no SCF
ISMEAR  = -5         # Tetrahedron for DOS
LORBIT  = 11
NEDOS   = 3001
LWAVE   = .FALSE.
```

Use a denser k-mesh (e.g., 16x16x16). Copy CHGCAR from Step 1. (PDF p. 122-123)

### Step 3: Band structure (non-SCF, line-mode k-points)

```
# INCAR for band structure
ICHARG  = 11         # Non-SCF from converged CHGCAR
ISMEAR  = 0          # Gaussian (tetrahedron not valid for line-mode)
SIGMA   = 0.1
LORBIT  = 11
LWAVE   = .FALSE.
LCHARG  = .FALSE.
```

Use a line-mode KPOINTS file (Section 3.3). Copy CHGCAR from Step 1. Band data is in EIGENVAL and PROCAR. (PDF p. 123)

### Key points
- The charge density and potential converge much faster than wavefunctions with increasing k-points. Generate a high-quality CHGCAR with a moderate k-mesh, then use ICHARG=11 for band/DOS runs. (PDF p. 123)
- For ICHARG=11, k-points are independent — the job can be split and merged later.
- Band structure and DOS require separate KPOINTS files: line-mode for bands, automatic mesh for DOS.

---

## 16. Molecular Dynamics

### INCAR settings for ab initio MD

```
IBRION  = 0          # MD (Verlet algorithm)
NSW     = 5000        # Number of MD steps
POTIM   = 1.0         # Time step in fs (MUST set explicitly)
SMASS   = 0           # Nose-Hoover thermostat (canonical NVT)
TEBEG   = 300         # Starting temperature (K)
TEEND   = 300         # Final temperature (K)
ISYM    = 0           # Turn off symmetry for MD
NELMIN  = 4           # Minimum electronic steps (ensure converged forces)
LWAVE   = .FALSE.     # Save disk
LCHARG  = .FALSE.     # Save disk
LREAL   = Auto        # Real-space projection for speed
```

### SMASS — Thermostat control

| Value | Ensemble | Notes |
|-------|----------|-------|
| -3 | NVE (microcanonical) | No thermostat |
| -1 | Linear velocity scaling | Rescale to ramp T from TEBEG to TEEND |
| 0 | Nose-Hoover chain (canonical NVT) | Standard canonical MD |
| >0 | Nose-Hoover with specified mass | Larger SMASS = slower coupling |

### Temperature correction
The kinetic temperature includes all degrees of freedom. For a finite system, the actual temperature is:

T_actual = T_set * N_ions / (N_ions - 1)

This correction matters for small cells. (PDF p. 61)

### PREC and ALGO for MD
- `PREC = Low` or `Normal` is adequate for most MD runs. High accuracy is not needed since the cell shape is constant. (PDF p. 116)
- Use `ALGO = VeryFast` (RMM-DIIS) for speed if wavefunctions are extrapolated between steps.
- Set `BMIX = 2.0` and `MAXMIX = 50` for stable extrapolation. Set `NBLOCK = 100` and `KBLOCK = 100` to write XDATCAR and PCDAT at regular intervals. (PDF p. 128)

### Simulated annealing
Set `TEBEG` and `TEEND` to different values to ramp the temperature linearly over the MD run. Use `SMASS = -1` for velocity scaling, or `SMASS = 0` (Nose-Hoover) for better sampling. Multiple restarts at decreasing temperatures help find global minima. (PDF p. 129)

---

## 17. Common Calculation Recipes

### 17.1 Static single-point energy

```
NSW     = 0
IBRION  = -1
ISMEAR  = -5       # Tetrahedron for semiconductors
EDIFF   = 1E-6
PREC    = Accurate
```

### 17.2 Ion-only relaxation

```
NSW     = 200
IBRION  = 2         # Conjugate gradient
ISIF    = 2         # Fix cell
EDIFFG  = -0.02     # Force criterion: 0.02 eV/Ang
PREC    = Accurate
ISMEAR  = 0         # Gaussian for relaxation
SIGMA   = 0.05
```

### 17.3 Full relaxation (ions + cell)

```
NSW     = 200
IBRION  = 2
ISIF    = 3         # Relax everything
ENCUT   = 520       # 1.3x default to reduce Pulay stress
EDIFFG  = -0.02
PREC    = Accurate
ISMEAR  = 0
SIGMA   = 0.05
```

### 17.4 Accurate equation of state

The safest way to get equilibrium volume and bulk modulus (PDF p. 115):

1. Pick several volumes; at each one, relax ions and cell shape at fixed volume (ISIF=4, ISMEAR=0 or 1).
2. Restart each from the previous CONTCAR (skip if cell shape is reasonable).
3. Do a final static run at each volume with ISMEAR=-5 (tetrahedron).
4. Fit E(V) to Birch-Murnaghan or Murnaghan equation of state.

### 17.5 Accurate bulk relaxation (3-step Pulay stress procedure)

When ISIF=3 gives wrong lattice parameters due to Pulay stress (PDF pp. 114-115):

1. **Step 1**: Relax with ISIF=3 at ENCUT = 1.3 * ENMAX (or higher). This gives an approximate volume.
2. **Step 2**: Take the CONTCAR, re-relax with ISIF=4 (constant volume) at several volumes near the Step 1 result.
3. **Step 3**: Do static calculations (IBRION=-1) at each volume with ISMEAR=-5 and fit to an equation of state.

### 17.6 Magnetic system (spin-polarized)

```
ISPIN   = 2
MAGMOM  = 4*5.0 8*0.6     # Initial moments per atom
ISMEAR  = 1                # Methfessel-Paxton for metals
SIGMA   = 0.1
LORBIT  = 11               # Projected moments in OUTCAR
```

### 17.7 Surface slab calculation

```
# Slab with vacuum
IDIPOL  = 3         # Dipole correction perpendicular to surface
LDIPOL  = .TRUE.    # Enable dipole correction
ISMEAR  = 0
SIGMA   = 0.05
# Fix bottom layers with Selective dynamics in POSCAR
```

See Section 27 for the full surface workflow.

### 17.8 Frequently used INCAR defaults (PDF pp. 96-105)

**Continuation from good WAVECAR/CHGCAR:**
```
ISTART = 1
ICHARG = 0
```

**Performance for large cells (>50 atoms):**
```
LREAL  = Auto
ALGO   = Fast
PREC   = Normal
```

**GGA from a bad initial guess:**
```
VOSKOWN = 1        # Use Vosko-Wilk-Nusair for GGA (improves atomization)
AMIX    = 0.2
BMIX    = 0.0001   # Safer mixing for poor starting charge
```

---

## 18. Output Files

| File | Contents | When written |
|------|----------|--------------|
| **OUTCAR** | Full log: parameters, energies, forces, stress, timing | Always |
| **OSZICAR** | Convergence per electronic/ionic step (compact) | Always |
| **CONTCAR** | Final structure (same format as POSCAR) | After ionic steps |
| **CHGCAR** | Charge density on FFT grid | LCHARG=.TRUE. |
| **CHG** | Charge density (lower precision, for visualization) | LCHARG=.TRUE. |
| **WAVECAR** | Wavefunctions (binary, large) | LWAVE=.TRUE. |
| **DOSCAR** | Total and projected DOS | After SCF |
| **EIGENVAL** | Eigenvalues and k-points | Always |
| **PROCAR** | Projected band character (lm-resolved) | LORBIT >= 10 |
| **XDATCAR** | Trajectory (ionic positions per step) | NSW > 0 |
| **LOCPOT** | Local potential on FFT grid | LVTOT=.TRUE. |
| **ELFCAR** | Electron localization function | LELF=.TRUE. |
| **IBZKPT** | Irreducible k-points and weights | Always |
| **vasprun.xml** | Full calculation data in XML | VASP 5+ |

### Useful OUTCAR grep patterns

```bash
grep "free  energy" OUTCAR          # Total energy (TOTEN)
grep "FORCES:" OUTCAR               # Check if forces are written
grep "entropy T*S" OUTCAR           # Smearing entropy (should be small)
grep "total drift" OUTCAR           # Force drift (wrap-around check)
grep "magnetization (x)" OUTCAR     # Total and local magnetic moments
grep "NELECT" OUTCAR                # Total electrons
grep "external pressure" OUTCAR     # Pressure for cell relaxation
grep "THz" OUTCAR                   # Phonon frequencies (IBRION=5/6/7/8)
grep "BORN EFFECTIVE CHARGES" OUTCAR  # Born charges (LEPSILON=.TRUE.)
```

---

## 19. Common Errors and Troubleshooting

### Electronic convergence failure
- **Symptoms**: NELM reached; energy oscillates in OSZICAR.
- **Fixes**: Reduce AMIX/BMIX (try AMIX=0.2, BMIX=0.0001, AMIX_MAG=0.8, BMIX_MAG=0.0001 for magnetic/slab systems). Switch ALGO to All. Increase NELM. Start from atomic charge (ICHARG=2). (PDF p. 74)

### Ionic convergence failure
- **Symptoms**: NSW reached; forces still large.
- **Fixes**: Reduce POTIM (0.1-0.2). Switch IBRION to 1 (quasi-Newton near minimum) or 3 (damped MD for difficult cases). Check if initial structure is reasonable.

### Wrap-around errors
- **Symptoms**: "WARNING: wrap around error must be expected" in OUTCAR. Jagged E-V curve.
- **Fixes**: Increase ENCUT. Use PREC=Accurate or High. Set FFT grids (NGX, NGY, NGZ) manually. (PDF pp. 117-118)
- **Check**: grep `total drift` in OUTCAR — drift should be << force magnitudes (typically < 0.1 eV/Angstrom).

### Pulay stress (volume relaxation artifacts)
- **Symptoms**: Equilibrium volume wrong; stress tensor unreliable at default cutoff.
- **Fixes**: Increase ENCUT to 1.3x default. Use ISIF=4 at fixed volumes + E(V) fit instead of ISIF=3. Add PSTRESS correction. See Section 17.5 for the 3-step procedure. (PDF pp. 114-115)

### ZBRENT warnings
- **Symptoms**: "ZBRENT: fatal error" during ionic relaxation.
- **Fixes**: Reduce POTIM. Improve initial structure. Switch to IBRION=3.

### Sub-space rotation errors (RMM-DIIS)
- **Symptoms**: "WARNING: Sub-Space-Matrix is not hermitian" or "internal error in EDDRMM".
- **Fixes**: Switch to ALGO=Normal or ALGO=All. The RMM-DIIS algorithm can be unstable for difficult systems. (PDF p. 71)

### Symmetry-related issues
- VASP may detect a higher symmetry than intended (especially after rounding atomic positions).
- Use `ISYM = 0` to disable symmetry for MD or very distorted cells.
- For non-collinear/SOC calculations, symmetry is often unreliable — use `ISYM = 0`. (PDF p. 90)

### Incorrect POTCAR ordering
- **Symptoms**: Wrong energies, forces make no sense, negative NBANDS.
- **Fix**: POTCAR species must match POSCAR species in the same order. Verify with `grep TITEL POTCAR`.

---

## 20. Convergence Testing

Always test convergence with respect to:

### ENCUT convergence
1. Fix KPOINTS to a reasonable value.
2. Run static calculations at increasing ENCUT (e.g., 200, 300, 400, 500, 600 eV).
3. Plot total energy vs. ENCUT. Converged when energy changes < 1-2 meV/atom.

### k-points convergence
1. Fix ENCUT to a reasonable value.
2. Run static calculations with increasing k-mesh density (e.g., 4x4x4, 6x6x6, 8x8x8, 10x10x10, 12x12x12).
3. Plot total energy vs. number of k-points. Converged when energy changes < 1-2 meV/atom.
4. **Note**: The error is not transferable between different lattice types (fcc vs. bcc vs. sc grids differ significantly at the same mesh density). (PDF p. 119)

### Recommended convergence targets
- Bulk total energy: < 1 meV/atom
- Forces: < 1 meV/Angstrom
- Stress tensor: converges slowly; may need 1.3-1.5x default ENCUT
- Band gap: typically converges faster than total energy

### ISMEAR consistency
- Use the **same** ISMEAR and SIGMA for all data points in a convergence test.
- For comparing energies: ISMEAR=-5 (tetrahedron) is fool-proof and parameter-free for semiconductors. (PDF p. 119)
- Even/odd k-mesh grids can give different k-point counts in the IRBZ, causing apparent convergence oscillations. Use even meshes for consistency. (PDF p. 120)

---

## 21. Hybrid Functionals

### HF/Hybrid calculations (VASP 5+)

```
LHFCALC = .TRUE.         # Enable Hartree-Fock exchange
ALGO    = Damped          # or All — Davidson/RMM-DIIS not valid for HF
TIME    = 0.4             # Reduce time step for stability (default 0.4)
AEXX    = 0.25            # Fraction of exact exchange (PBE0 default)
AGGAX   = 0.75            # GGA exchange fraction (= 1 - AEXX for PBE0)
AGGAC   = 1.0             # GGA correlation fraction
ALDAC   = 0.0             # LDA correlation fraction
```

### PBE0

25% exact exchange + 75% PBE exchange + 100% PBE correlation. Set AEXX=0.25 (default). (PDF p. 94)

### HSE06 — Screened hybrid (VASP 5+)

```
LHFCALC  = .TRUE.
HFSCREEN = 0.2            # Screening parameter (1/Angstrom); 0.2 = HSE06
ALGO     = Damped          # or All
AEXX     = 0.25
AGGAX    = 0.75
AGGAC    = 1.0
ALDAC    = 0.0
PRECFOCK = Fast            # or Normal/Accurate; Fast for routine use
```

- **HSE03**: HFSCREEN = 0.3 (original value from Heyd, Scuseria, Ernzerhof).
- **HSE06**: HFSCREEN = 0.2 (revised, recommended). The "06" value is widely adopted.
- NKRED (or NKREDX/NKREDY/NKREDZ): Reduces the k-point grid for the exact-exchange kernel. Set NKRED=2 to speed up HSE calculations with minimal accuracy loss. EVENONLY/ODDONLY further reduce the grid. (PDF pp. 100-101)
- `ENCUTFOCK = 0`: Accelerate HF with a reduced FFT grid (factor 2-3x speedup, small noise in forces). (PDF p. 94)
- **Version note**: HSE06 is not in the 2005 PDF; see https://vasp.at/wiki/HFSCREEN

### Berry phase polarization (VASP 4.5+)

```
LBERRY  = .TRUE.
IGPAR   = 1              # Direction (1, 2, or 3)
NPPSTR  = 6              # Points on k-string
DIPOL   = 0.25 0.25 0.25 # Center of cell
```

Calculates electronic polarization via the Berry phase. Requires separate runs for each Cartesian direction (IGPAR=1,2,3). (PDF pp. 86-88)

### NEB / Elastic Band (VASP 4.X+)

```
IMAGES  = 5              # Number of intermediate images
SPRING  = -5             # Spring constant (negative = NEB)
IBRION  = 1              # or 3 (damped MD, recommended)
```

Each image in a subdirectory (00/, 01/, ..., 06/) with its own POSCAR. Endpoints (00/ and 06/) are fixed. Total MPI ranks must be divisible by IMAGES. (PDF p. 79)

---

## 22. Meta-GGA Functionals (VASP 5.3+)

Meta-GGA functionals depend on the kinetic energy density in addition to the density and its gradient. Set with the METAGGA tag. See: https://vasp.at/wiki/METAGGA

### METAGGA values

| Value | Functional | VASP version | Notes |
|-------|-----------|-------------|-------|
| TPSS | Tao-Perdew-Staroverov-Scuseria | 5.3+ | First meta-GGA in VASP |
| RTPSS | Revised TPSS | 5.3+ | |
| M06L | Minnesota M06-L | 5.3+ | |
| MS0, MS1, MS2 | Made Simple 0/1/2 | 5.3+ | |
| MBJ | Modified Becke-Johnson | 5.3+ | Potential-only; no total energy/forces |
| LMBJ | Local MBJ | 5.3+ | Potential-only; no total energy/forces |
| **SCAN** | **Strongly Constrained and Appropriately Normed** | **5.4.4+** | **Recommended meta-GGA for solids** |
| RSCAN | Regularized SCAN | 6.2.0+ | Numerically smoother than SCAN |
| **R2SCAN** | **r2SCAN** | **6.2.0+** | **Recommended: accurate and numerically stable** |
| SCANL, RSCANL, R2SCANL | Deorbitalized variants | 6.4.0+ | Faster; no kinetic energy density |
| TASK_X | TASK exchange | 6.5.0+ | |
| LAK | LAK functional | 6.5.0+ | |
| LIBXC | Use Libxc library | 6.3.0+ | Access to 400+ functionals |

### Key settings for meta-GGA

```
METAGGA = R2SCAN
LASPH   = .TRUE.          # Strongly recommended — non-spherical contributions
ALGO    = All              # Conjugate gradient; meta-GGAs can be sensitive
PREC    = Accurate
```

### Important notes
- **LASPH = .TRUE.** is strongly recommended for all meta-GGA calculations. Without it, errors of several meV/atom can occur.
- **MBJ and LMBJ** are potential-only functionals — they give band gaps but no total energy, forces, or stress. Cannot be used for relaxation.
- **Convergence tip**: Pre-converge with PBE (GGA=PE) first, then restart with the meta-GGA (copy WAVECAR and CHGCAR).
- Standard PBE POTCARs are appropriate for meta-GGA calculations.
- For SCAN and r2SCAN, use `ALGO = All` if `ALGO = Fast` shows instability.

---

## 23. Van der Waals Corrections

Three approaches for including van der Waals dispersion in VASP: pairwise corrections (DFT-D), nonlocal correlation functionals (vdW-DF), and the Tkatchenko-Scheffler methods.

### 23.1 DFT-D3: Grimme D3 correction

Set via the IVDW tag. See: https://vasp.at/wiki/DFT-D3

| IVDW | Method | Notes |
|------|--------|-------|
| 11 | DFT-D3 zero-damping | Original Grimme D3 |
| 12 | DFT-D3 BJ-damping | **Recommended**: Becke-Johnson damping |

```
# PBE + D3(BJ)
GGA   = PE
IVDW  = 12
```

Default parameters are set automatically for common functionals (PBE, RPBE, revPBE, PBEsol, TPSS, M06L, SCAN, B3LYP, PBE0, HF). Override with: VDW_S6, VDW_S8, VDW_A1, VDW_A2, VDW_SR, VDW_SR8, VDW_RADIUS, VDW_CNRADIUS.

### 23.2 DFT-D4 (VASP 6.6.0+)

```
IVDW = 13
```

Tags: DFTD4_MODEL (default=d4), DFTD4_XC (auto-detected from GGA/METAGGA), VDW_S6, VDW_S8, VDW_A1, VDW_A2, VDW_S9, VDW_RADIUS, VDW_CNRADIUS. See: https://vasp.at/wiki/DFT-D4

### 23.3 Nonlocal vdW-DF functionals

These functionals add a nonlocal correlation energy term. See: https://vasp.at/wiki/Nonlocal_vdW-DF_functionals

```
LUSE_VDW = .TRUE.         # Enable nonlocal vdW
LASPH    = .TRUE.          # Recommended
AGGAC    = 0.0             # Zero GGA correlation (replaced by vdW)
```

| Functional | GGA | Extra tags | Notes |
|------------|-----|-----------|-------|
| vdW-DF | RE | | Original Dion et al. |
| vdW-DF2 | ML | ZAB_VDW = -1.8867 | Lee et al. |
| optPBE-vdW | OR | | Klimes et al.; good for surfaces |
| optB88-vdW | BO | PARAM1=0.1833333333, PARAM2=0.22 | Klimes et al. |
| optB86b-vdW | MK | PARAM1=0.1234, PARAM2=1.0 | Klimes et al. |
| **rev-vdW-DF2** | MK | PARAM1=0.1234568, PARAM2=0.7114, ZAB_VDW=-1.8867 | **Hamada; widely recommended** |
| SCAN+rVV10 | — | METAGGA=SCAN, IVDW_NL=2, BPARAM=..., CPARAM=... | Meta-GGA + vdW |
| r2SCAN+rVV10 | — | METAGGA=R2SCAN, IVDW_NL=2 | Meta-GGA + vdW |

**Version notes**:
- vdW-DF energies and forces: VASP 5.2.12+ (May 2011). Stress: 5.2.12+ (Nov 2011 for non-spin, 5.3.1 for spin).
- Before VASP 6.4.0, `vdw_kernel.bindat` must be in the run directory. From 6.4.3, it is generated automatically.
- IVDW_NL tag (select kernel explicitly) introduced in VASP 6.4.0.
- rVV10 stress was broken before 6.3.0.

---

## 24. Optical Properties and Dielectric Response (VASP 5+)

### Frequency-dependent dielectric function

```
LOPTICS = .TRUE.           # Compute dielectric function
CSHIFT  = 0.1              # Complex shift (eV); default 0.1
NEDOS   = 2000             # Energy grid points
NBANDS  = (increase)       # Need many empty bands for optical transitions
ISMEAR  = -5               # Tetrahedron recommended
```

Output: real and imaginary parts of the dielectric tensor in OUTCAR and vasprun.xml. Optical absorption, reflectivity, and refractive index can be derived from the dielectric function. (PDF pp. 97-99)

### LNABLA — Transverse gauge

- `LNABLA = .TRUE.`: Uses the nabla (transverse) gauge for optical matrix elements instead of the default longitudinal gauge. May improve accuracy for some systems.

### Static dielectric properties via DFPT

```
LEPSILON = .TRUE.           # Calculate static dielectric tensor via DFPT
LRPA     = .TRUE.           # Random Phase Approximation (optional)
```

LEPSILON uses density-functional perturbation theory to compute the static dielectric tensor and Born effective charges. This is independent of LOPTICS (which uses independent-particle approximation). (PDF p. 98)

---

## 25. GW Calculations (VASP 5+)

Many-body perturbation theory in the GW approximation. See: https://vasp.at/wiki/GW_calculations

### Two-step G0W0 recipe (PDF pp. 99-100)

**Step 1: Standard DFT ground state**

```
# Converged SCF with sufficient empty bands
NBANDS  = (4-8x occupied)   # Many empty bands needed
ISMEAR  = -5
PREC    = Accurate
LWAVE   = .TRUE.             # Need WAVECAR for GW
LCHARG  = .TRUE.
```

**Step 2: GW calculation**

```
ALGO    = GW0                # or GW for single-shot G0W0
NELM    = 1                  # Single iteration for G0W0
NOMEGA  = 50                 # Frequency grid points (50-100 typical)
LSPECTRAL = .TRUE.           # Use spectral method (recommended, faster)
ENCUTGW = (0.5-0.7)*ENCUT   # Can be 30-50% smaller than ENCUT
PRECFOCK = Fast              # Speed up Fock exchange evaluation
```

### Key GW tags

| Tag | Default | Purpose |
|-----|---------|---------|
| NOMEGA | 50 | Frequency points for self-energy |
| NOMEGAR | 50 | Real-frequency points |
| LSPECTRAL | .FALSE. | Spectral representation (set .TRUE. for speed) |
| OMEGAMAX | auto | Maximum frequency |
| OMEGATL | auto | Transition frequency between grid types |
| ENCUTGW | 2/3*ENCUT | Response function cutoff |

### HSE06 + GW recipe (PDF p. 101)

For improved starting point:

1. Converge HSE06 ground state (Section 21).
2. Read HSE06 WAVECAR: set ISTART=1.
3. Run GW on top of the hybrid starting point.

**Caution**: GW calculations are very expensive (memory and time scale as NBANDS^2-3). Start with small cells and test NOMEGA, ENCUTGW, and NBANDS convergence carefully.

---

## 26. Phonons and Lattice Dynamics

Two approaches in VASP: finite differences (IBRION=5/6) and DFPT (IBRION=7/8). Both compute zone-center (Gamma-point) phonon frequencies directly. For full phonon dispersions, use a supercell approach with post-processing by phonopy. See: https://vasp.at/wiki/Phonons_from_finite_differences and https://vasp.at/wiki/Phonons_from_density-functional-perturbation_theory

### 26.1 Finite differences (IBRION=5/6)

```
IBRION  = 6              # Symmetry-reduced (or 5 for all atoms)
NFREE   = 2              # Central differences (2 displacements per direction)
POTIM   = 0.015          # Displacement in Angstrom
PREC    = Accurate        # Accurate forces required
EDIFF   = 1E-8           # Tight electronic convergence
ISMEAR  = 0
SIGMA   = 0.05
NSW     = 1              # VASP handles the loop internally
```

- **IBRION=5**: Displaces every atom in all 3 Cartesian directions. Expensive but general.
- **IBRION=6**: Uses symmetry to reduce displacements (VASP 5.1+). Recommended.
- **NFREE=2**: Central differences (default, recommended). NFREE=4 uses 4 displacements per direction for higher accuracy.
- **ISIF >= 3**: Also computes elastic constants and internal strain tensors with IBRION=6.
- Works with any functional (GGA, meta-GGA, hybrid, DFT+U).

### 26.2 DFPT (IBRION=7/8)

```
IBRION   = 8             # Symmetry-reduced DFPT (or 7 for all atoms)
PREC     = Accurate
EDIFF    = 1E-8
ISMEAR   = 0
SIGMA    = 0.05
LEPSILON = .TRUE.         # Also compute Born charges and dielectric tensor
```

- **IBRION=7**: All atoms. **IBRION=8**: Symmetry-reduced (VASP 5.1+).
- DFPT removes the need to choose the displacement magnitude (POTIM).
- **Limitation**: Only works with LDA and GGA functionals. Not available for meta-GGA, hybrid, or DFT+U.
- No elastic tensors from DFPT (strain perturbation not implemented).
- Born effective charges are computed analytically when LEPSILON=.TRUE.

### Phonon dispersions (supercell approach)

1. Build a supercell (e.g., 2x2x2 or 3x3x3 of the primitive cell).
2. Run finite differences (IBRION=5 or 6) in the supercell.
3. Reduce the k-mesh proportionally (e.g., 12x12x12 primitive → 6x6x6 for 2x2x2 supercell).
4. Post-process with phonopy to Fourier-interpolate the dynamical matrices and obtain the full dispersion.

### Checking phonon output

```bash
grep "THz" OUTCAR     # Frequencies; "f/i" = imaginary (soft mode)
```

Each mode is labeled "f" (real frequency) or "f/i" (imaginary — indicates structural instability). The last 3 modes are acoustic (translational) and should be near zero.

---

## 27. Surface Calculations

A five-step procedure for converged surface calculations (PDF pp. 129-131):

### Step 1: Bulk calculation

Fully converge the bulk structure (lattice parameter, k-mesh, ENCUT). Record the converged lattice constant and the FFT mesh sizes (NGX, NGY, NGZ) from OUTCAR.

### Step 2: Set FFT grids for the slab

Set NGX, NGY, NGZ explicitly in INCAR for the slab calculation. The in-plane grids should match the bulk proportionally; the perpendicular grid should be at least as dense. This ensures consistent FFT grids between bulk and slab.

### Step 3: Build the slab

- Choose the surface facet (e.g., (001), (110), (111)).
- Include enough bulk-like layers (typically 5-9 for metals, 4-7 for oxides).
- Add sufficient vacuum (12-20 Angstrom, converge this).
- Fix the bottom 2-3 layers using Selective Dynamics in POSCAR.

### Step 4: Relax the surface

```
IBRION  = 2
ISIF    = 2              # Fix cell shape and volume
EDIFFG  = -0.02
PREC    = Accurate
IDIPOL  = 3              # Dipole correction along z (perpendicular to surface)
LDIPOL  = .TRUE.
ISMEAR  = 0
SIGMA   = 0.05
# Set NGX, NGY, NGZ from Step 2
```

### Step 5: Convergence checks

Test convergence with respect to:
- Number of slab layers (surface energy should converge)
- Vacuum thickness (total energy should not change)
- k-mesh density in the surface plane
- ENCUT

### Surface energy

E_surf = (E_slab - N * E_bulk) / (2 * A)

where N is the number of formula units in the slab, E_bulk is the bulk energy per formula unit, and A is the surface area. The factor 2 accounts for two surfaces. For asymmetric slabs (one side relaxed, one fixed), this formula gives an average surface energy.

### Work function

Set `LVTOT = .TRUE.` to write LOCPOT. The work function is:

phi = V_vacuum - E_Fermi

Extract V_vacuum from the plateau in the electrostatic potential (LOCPOT) in the vacuum region.

---

## 28. POTCAR Selection Guide

Detailed element-by-element recommendations from the VASP manual (PDF pp. 131-135) and wiki.

### Recommended PAW potentials by element

| Element | Recommended | ENMAX (approx.) | Notes |
|---------|------------|-----------------|-------|
| H | H | 250 | `H_h` for high accuracy; `H_s` for fast tests |
| Li | Li_sv | 271 | 1s in valence; `Li` misses core-valence interaction |
| Be | Be_sv | 309 | 1s in valence |
| B | B | 319 | |
| C | C | 400 | `C_h` for high accuracy |
| N | N | 400 | `N_s` for large cells |
| O | O | 400 | `O_s` for fast tests; `O_h` for precise forces |
| F | F | 400 | |
| Na | Na_pv | 260 | p-valence essential |
| Mg | Mg | 200 | `Mg_pv` if p states matter |
| Al | Al | 240 | |
| Si | Si | 245 | |
| P | P | 255 | |
| S | S | 259 | |
| Cl | Cl | 262 | |
| K | K_sv | 259 | s-valence essential |
| Ca | Ca_pv | 267 | p-valence for oxide chemistry |
| Sc-Zn | Standard or _pv | 200-300 | Use `_pv` for early 3d (Sc-V) |
| Ga | Ga_d | 283 | d-electrons for III-V semiconductors |
| Ge | Ge_d | 310 | d-electrons for accuracy |
| Sr | Sr_sv | 229 | s-valence |
| Ba | Ba_sv | 187 | s-valence |
| La-Lu | Standard or _3 | varies | `_3` treats f as core (no DFT+U needed) |

**General rules**:
- For alkali metals: always use `_sv` (K, Rb, Cs) or `_pv` (Na).
- For alkaline earth metals: use `_pv` (Ca, Mg when p states matter) or `_sv` (Sr, Ba).
- For transition metals: standard is usually fine; `_pv` for early 3d when semi-core p participates in bonding.
- For rare earths: use `_3` variant if not doing DFT+U (treats f as core).
- Always verify with `grep ENMAX POTCAR` and convergence tests.

---

## 29. Machine-Learned Force Fields (VASP 6.3+)

VASP can train and deploy machine-learned force fields (MLFF) on-the-fly during ab initio MD. See: https://vasp.at/wiki/ML_LMLFF and https://vasp.at/wiki/Machine_learning_force_field_calculations

### Key tags

| Tag | Value | Purpose |
|-----|-------|---------|
| ML_LMLFF | .TRUE. | Enable MLFF (master switch; all ML_ tags ignored if .FALSE.) |
| ML_MODE | train | On-the-fly training during MD |
| ML_MODE | refit | Fast prediction refit (VASP 6.4.0+; 20-100x speedup) |
| ML_MODE | run | Production with trained force field |

### Training workflow

**Step 1: On-the-fly training**

```
# Standard MD INCAR + MLFF tags
IBRION  = 0
NSW     = 5000
POTIM   = 2.0
SMASS   = 0
TEBEG   = 300
TEEND   = 300
ISYM    = 0
ML_LMLFF = .TRUE.
ML_MODE  = train
LREAL   = Auto
LWAVE   = .FALSE.
LCHARG  = .FALSE.
```

VASP runs ab initio MD and simultaneously trains the MLFF. When the MLFF is confident, it uses MLFF predictions instead of DFT (speeding up the run). Outputs: ML_ABN (training database), ML_FFN (force field binary), ML_LOGFILE (training log).

**Step 2: Refit (VASP 6.4.0+)**

```bash
cp ML_ABN ML_AB
```

```
ML_LMLFF = .TRUE.
ML_MODE  = refit
```

This creates an optimized ML_FFN for fast prediction. Strongly recommended before production.

**Step 3: Production**

```bash
cp ML_FFN ML_FF
```

```
ML_LMLFF = .TRUE.
ML_MODE  = run
```

Only MLFF predictions are used — no DFT. MPI-only parallelization (no GPU/OpenMP for run/refit modes).

### Training tips
- Train over the temperature and pressure range you plan to study.
- Continue training by copying ML_ABN to ML_AB and restarting with ML_MODE=train.
- Check ML_LOGFILE for training errors (lines starting with `ERR`).
- ML_FFN files are usually backward-compatible between VASP versions.

---

## Version Compatibility Notes

This skill covers tags and defaults from two sources. When they conflict, both are noted:

| Feature | VASP 4.4-4.6 (PDF, 2005) | VASP 5/6 (wiki) |
|---------|--------------------------|------------------|
| PREC Normal/Accurate | Introduced in 4.5 | Standard |
| ALGO Fast/VeryFast | Introduced in 4.5 | Standard |
| GGA = PE (PBE) | Introduced in 4.5 | Standard |
| Species line in POSCAR | Not required | Required (5+) |
| NCORE | Not available | Available (5.2+) |
| KPAR | Not available | Available (5+) |
| LSORBIT (SOC) | Late beta (4.5+) | Stable |
| LNONCOLLINEAR | Late beta (4.5+) | Stable |
| LDAU (DFT+U) | Late beta (4.6+) | Stable |
| LHFCALC (hybrids) | VASP 5 only, internal docs | Stable |
| HSE06 (HFSCREEN) | Not in PDF | Available (5+) |
| METAGGA (SCAN etc.) | Not in PDF | Available (5.3+; SCAN 5.4.4+) |
| IVDW (DFT-D3/D4) | Not in PDF | D3: 5+; D4: 6.6.0+ |
| LUSE_VDW (vdW-DF) | Not in PDF | Available (5.2.12+) |
| LOPTICS | Not in PDF | Available (5+) |
| IBRION=6 (symm. FD) | Not available | Available (5.1+) |
| IBRION=7/8 (DFPT) | Not available | Available (5.1+) |
| ML_LMLFF | Not available | Available (6.3+) |
| vasprun.xml | Not documented | Standard output (5+) |
| IMAGES (NEB) | Available (4.X+) | Available |

For modern VASP 5/6 features not in the 2005 PDF, consult the official wiki: https://vasp.at/wiki/The_VASP_Manual
