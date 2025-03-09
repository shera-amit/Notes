Here's a comprehensive and structured guide for troubleshooting non-converging **VASP calculations**, covering single-point calculations, cell and ion relaxations, and similar tasks. These guidelines will help you identify, diagnose, and fix common convergence issues systematically.

---

## ⚠️ **Pre-check (Common for all calculation types)**

Before rerunning, **always** verify:

- **POSCAR structure**: Check atomic positions, lattice constants, symmetry, and vacuum spacing (for slabs/molecules).
- **POTCAR suitability**: Ensure correct pseudopotentials are chosen (PAW recommended).
- **INCAR settings**: Verify calculation settings match your goal (accuracy vs. speed trade-off).
- **KPOINTS sampling**: Convergence critically depends on sufficient and appropriate k-point density.

---

## 🔍 **General Troubleshooting for Convergence Issues (Electronic SCF)**

### **1. Electronic convergence parameters:**
- **NELM**: Increase the maximum number of electronic iterations (`NELM=100` or more).
- **EDIFF**: Adjust the convergence criteria slightly looser (`EDIFF=1E-05`) or tighter (`EDIFF=1E-06`) depending on your goal.
- **ALGO**: Change algorithm (`Normal`, `Fast`, `All`, `Damped`) to improve convergence.
  - **ALGO=Normal** → Davidson diagonalization (default)
  - **ALGO=Fast** → Faster but may not always converge
  - **ALGO=All** → More stable but slower
  - **ALGO=Damped** → Good for difficult-to-converge systems (metals, magnetic systems)

### **2. Mixing scheme adjustments:**
- **AMIX, BMIX, AMIX_MAG, BMIX_MAG**: Lower `AMIX` and `BMIX` to `0.02`-`0.1` (slower but more stable mixing).
- **IMIX**: Change mixing algorithm (`IMIX=0` default, `IMIX=1` for Kerker, `IMIX=4` for Pulay), often:
  ```bash
  IMIX = 1
  AMIX = 0.02
  BMIX = 0.0001
  ```
- **Mixing history** (`MAXMIX`): Set `MAXMIX=40-60` for difficult convergence.

### **3. Occupancy and smearing:**
- **ISMEAR**: Test different smearing methods:
  - For metals: `ISMEAR=1` (Methfessel-Paxton), `ISMEAR=-5` (tetrahedron method, better for DOS)
  - For semiconductors/insulators: `ISMEAR=0` or `-5`
- **SIGMA**: Adjust broadening value (`SIGMA=0.01-0.2 eV`), lower sigma helps accuracy but can hurt convergence for metals.

### **4. Initial guess and charge density:**
- **ICHARG=2**: Restart from scratch.
- **ICHARG=1**: Restart from WAVECAR/CHGCAR files of previous converged run.
- For very challenging cases, consider breaking your calculation into steps:
  - Run with lower ENCUT and higher SIGMA, converge, then restart with standard ENCUT and SIGMA.

---

## 🔬 **Specific Calculation Type Convergence Tricks**

### 🚩 **1. Single-point (Static) calculations**
Goal: Obtain converged energy, electronic density, and electronic structure for fixed atomic and lattice positions.

- **ENCUT**: Perform convergence tests for ENCUT (e.g., `ENCUT=400-600 eV`).
- **KPOINTS**: Increase sampling density until energy changes less than ~1-2 meV/atom.
- **ISPIN**: Ensure correct spin polarization settings (`ISPIN=1` non-spin polarized, `ISPIN=2` spin polarized).
- **Magnetic initialization**:
  - If magnetic, carefully set MAGMOM to suitable values:
  ```bash
  ISPIN = 2
  MAGMOM = 2.0 2.0 -2.0 -2.0 ...
  ```
- If you suspect magnetic frustration: use different MAGMOM seeds.

---

### 🚩 **2. Ionic relaxation (IONS only, fixed cell shape and volume)**
Goal: Obtain a relaxed structure (atomic positions optimized, lattice constants fixed).

- **EDIFFG**: Set criteria for ionic convergence:
  - `EDIFFG=-0.02` eV/Å is standard. Reduce slightly if forces don't converge (`-0.01` to `-0.005`).
- **IBRION, POTIM**:
  - `IBRION=2` (conjugate-gradient, default), good for most cases.
  - `IBRION=1` (quasi-Newton), robust but slower.
  - `POTIM`: Decrease step size (`0.2-0.3`) for unstable optimizations.

- **Selective Dynamics**:
  - Fix problematic atoms temporarily, relax other atoms first, then re-enable relaxation.
  - Useful when certain atoms or adsorbates cause instability.

- **NSW**: Increase the max number of ionic steps (`NSW=100` or higher).

---

### 🚩 **3. Cell + Ionic relaxation (full relaxation: positions, shape, volume)**
Goal: Fully relaxed structure with optimized lattice constants, shape, and atomic positions.

- **ISIF parameter**:
  - `ISIF=3` → Full relaxation (ionic positions + cell shape & volume).
  - `ISIF=4` → Volume relaxation with fixed shape.
- **ENCUT**:
  - Typically higher ENCUT recommended to accurately capture stress tensor for relaxation.
- **IBRION and POTIM**:
  - `IBRION=2` recommended with smaller POTIM (`POTIM=0.2-0.4`) for full cell relaxation.
  - Consider multiple-step relaxations:
    - First relax ions only (`ISIF=2`).
    - Then relax shape/volume (`ISIF=3`) using previously relaxed structure as initial guess.

- **Symmetry and constraints**:
  - Check symmetry carefully. Break symmetry slightly (0.01 Å perturbation) to escape convergence loops.
  - Use selective dynamics or symmetry constraints (`ISYM`) judiciously.

---

## 🧑‍🔬 **Advanced Checks and Techniques:**

- **Spin-Orbit Coupling (SOC)**:
  - If included (`LSORBIT=.TRUE.`), first converge without SOC, then restart with SOC.

- **Van der Waals interactions (vdW)**:
  - Use vdW corrections (`IVDW=11, 12` (DFT-D3), `IVDW=20,21` (Tkatchenko-Scheffler)) for layered or molecular systems.

- **DFT+U Calculations**:
  - Carefully tune `LDAU` parameters (`LDAUU`, `LDAUJ`, `LDAUL`) for correlated systems.

- **Hybrid Functionals** (HSE06, PBE0):
  - Use standard PBE first, then use WAVECAR/CHGCAR from PBE as initial guess (`ICHARG=1`).

---

## 📊 **Monitoring and Diagnostics Tools:**
- Regularly check outputs (`OUTCAR`, `OSZICAR`) for signs of convergence:
  - Electronic convergence: total energy fluctuations, `dE`.
  - Ionic convergence: Forces, stress tensor (`grep stress OUTCAR`).
- Visualize structures regularly (`VESTA`, `ASE viewer`, `OVITO`).
- Check DOS and bands frequently for any anomaly indicating problematic convergence.

---

## 🚨 **What to avoid:**
- Avoid overly tight EDIFF early on—this often slows down convergence unnecessarily.
- Don't blindly increase ENCUT or KPOINTS without systematic convergence tests—this wastes computation time.

---

## ✅ **Final Recommendation Workflow:**

1. **Initial run** → quick and rough parameters.
2. **Monitor and identify problems** → `grep EDIFFG OUTCAR`, check forces, energies, stress.
3. **Adjust parameters incrementally** → mixing (`AMIX`, `BMIX`), smearing, relaxation settings (`POTIM`, `IBRION`, `ISIF`), restart strategies (`ICHARG`).
4. **Iteratively rerun** until stable and fully converged.

This systematic approach ensures you quickly pinpoint and overcome typical convergence hurdles in VASP calculations.
