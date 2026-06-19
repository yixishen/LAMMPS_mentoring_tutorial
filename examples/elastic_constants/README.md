# Worked Example: Elastic Constants

This exercise teaches students how to calculate elastic constants with LAMMPS
and why the result is one of the first checks to run before using a potential
for production simulations.

Start with the included example in `runnable_lj_fcc/`. It is an original,
compact teaching example that uses a Lennard-Jones FCC crystal, so students can
run it without downloading a separate potential file. The official LAMMPS
`examples/ELASTIC` directory is still a useful reference once students
understand the flow. The LAMMPS documentation describes the zero-temperature
method as deforming the simulation box in small strain directions with
`change_box` and measuring the resulting stress response. The
finite-temperature examples live in `examples/ELASTIC_T` and require more
careful averaging.

## Learning Goals

After this exercise, students should be able to:

- Run the included `runnable_lj_fcc` example.
- Understand what the official `ELASTIC` example is doing when they see it
  later.
- Explain what an elastic constant means physically.
- Identify `C11`, `C12`, and `C44` for a cubic crystal.
- Convert pressure units when needed.
- Modify the example to test a Ni or Ni-Cr EAM potential.
- Decide whether an interatomic potential is reasonable enough for later
  defect, stacking-fault, or grain-boundary simulations.

## Background

Elastic constants describe how stress changes when a material is strained. In
Voigt notation, the fourth-rank elastic tensor is written as a 6 by 6 matrix.
For a cubic material, symmetry reduces the independent constants to:

- `C11`: normal stiffness along a crystal axis.
- `C12`: coupling between normal strain and perpendicular normal stress.
- `C44`: shear stiffness.

Useful cubic checks:

```text
Bulk modulus:       B = (C11 + 2*C12)/3
Shear anisotropy:   A = 2*C44/(C11 - C12)
Mechanical stability: C11 > |C12|, C44 > 0, C11 + 2*C12 > 0
```

In `units metal`, pressure is reported in bar unless the input script converts
it. Since `1 GPa = 10000 bar`, many LAMMPS elastic scripts multiply stresses by
`1.0e-4` to report elastic constants in GPa. Always check the conversion factor
inside the script before comparing to literature values.

## Step 1: Run The Included Example

From the repository root:

```bash
cd examples/elastic_constants/runnable_lj_fcc
lmp -in in.relax_lj
lmp -in in.c11_c12_lj
lmp -in in.c44_lj
```

If your LAMMPS executable has another name, replace `lmp` with that name:

```bash
lammps_serial -in in.relax_lj
```

## Step 2: Inspect The Files

Before running, list the files:

```bash
ls
```

Typical files include:

- `in.relax_lj`: creates and relaxes a small FCC Lennard-Jones crystal.
- `in.c11_c12_lj`: applies a small x-direction strain and estimates `C11` and
  `C12`.
- `in.c44_lj`: applies a small shear strain and estimates `C44`.
- `README.md`: explains what each script does.

These scripts are intentionally short. Students should be able to identify the
initialization, structure creation, potential, minimization, deformation, and
output sections.

## Step 3: Understand The Zero-Temperature Calculation

The included example uses three steps:

1. Relax the initial crystal so the pressure is close to zero.
2. Apply a small strain with `change_box`.
3. Minimize again and calculate elastic constants from stress change divided by
   strain.

This is a teaching calculation, not a publication-quality elastic workflow.
The official `examples/ELASTIC` and `examples/ELASTIC_T` directories show more
complete approaches.

## Step 4: Record The Result

For each run, record:

- LAMMPS version from `lmp -help`.
- Operating system and whether the run used serial or MPI LAMMPS.
- Potential name and file.
- Crystal structure and lattice constant.
- Units reported by the script, usually GPa after conversion.
- Full elastic matrix if printed.
- For cubic crystals: `C11`, `C12`, `C44`, `B`, and `A`.

Example result table:

| Quantity | Value | Unit | Note |
| --- | ---: | --- | --- |
| `C11` |  | GPa | normal stiffness |
| `C12` |  | GPa | normal coupling |
| `C44` |  | GPa | shear stiffness |
| `B` |  | GPa | `(C11 + 2*C12)/3` |
| `A` |  | none | `2*C44/(C11 - C12)` |

## Step 5: Adapt To A Ni EAM Potential

Make a copy of the included starter example:

```bash
mkdir -p ~/lammps-work/elastic_constants
cp -r examples/elastic_constants/runnable_lj_fcc ~/lammps-work/elastic_constants/Ni_EAM
cd ~/lammps-work/elastic_constants/Ni_EAM
```

Then edit the structure and potential settings. The important LAMMPS changes
look like this:

```lammps
units           metal
atom_style      atomic
boundary        p p p

lattice         fcc 3.52

pair_style      eam/alloy
pair_coeff      * * /path/to/Ni_potential.eam.alloy Ni
```

Checklist:

- Use `units metal` for metallic EAM potentials.
- Use the correct lattice type, usually `fcc` for Ni.
- Start with a reasonable lattice constant, then compare with your lattice
  constant validation result.
- Make sure the `pair_coeff` element order matches LAMMPS atom types.
- Keep the deformation magnitude small enough for linear elasticity.
- Compare output with literature values or potential documentation, not just
  with another simulation.

## Step 6: Adapt To A Binary Ni-Cr Potential

For a binary system, the potential line may look like:

```lammps
pair_style      eam/alloy
pair_coeff      * * /path/to/NiCr_potential.eam.alloy Ni Cr
```

or, for a hybrid potential:

```lammps
pair_style      hybrid/overlay eam/alloy eam/fs
pair_coeff      * * eam/alloy /path/to/FeCrNi_d.eam.alloy Cr Ni
pair_coeff      * * eam/fs    /path/to/FeCrNi_s.eam.fs    Cr Ni
```

Important: the element list after `pair_coeff` must match atom types in the
simulation box. If type 1 is Cr and type 2 is Ni, use `Cr Ni`, not `Ni Cr`.

For a random alloy, elastic constants depend on composition and atomic
configuration. A serious workflow should average over several random seeds and
possibly several cell sizes.

## Step 7: Interpret The Result

A result is suspicious when:

- `C44` is negative.
- `C11 + 2*C12` is negative.
- The values differ wildly from known experimental or potential-reference
  values.
- A tiny change in deformation size causes a large change in the constants.
- The result changes strongly with cell size for a simple crystal.

This is not just bookkeeping. If the potential gives unreasonable elastic
constants, it may also give unreliable grain-boundary energies, stacking-fault
energies, dislocation behavior, or thermal expansion.

## Step 8: Finite-Temperature Elastic Constants

After the zero-temperature example works, inspect the official `ELASTIC_T`
examples. The LAMMPS documentation describes two provided approaches:

- `ELASTIC_T/BORN_MATRIX`: uses stress fluctuations and the Born matrix.
- `ELASTIC_T/DEFORMATION`: measures stress response after finite deformation
  during NVT simulations.

Finite-temperature calculations require equilibration, time averaging, and
uncertainty estimates. Do not treat a single short run as a production-quality
elastic constant.

## Student Deliverable

Submit:

- The edited input files.
- The `log.lammps` file.
- A table of `C11`, `C12`, `C44`, `B`, and `A`.
- A short paragraph explaining whether the potential looks reasonable.
- One note about what would be needed to make the calculation more reliable.

## References

- LAMMPS elastic constants howto:
  <https://docs.lammps.org/Howto_elastic.html>
- LAMMPS example scripts:
  <https://docs.lammps.org/Examples.html>
