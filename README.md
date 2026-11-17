# Unidirectional-UD-carbon-layup-Composite-Wing-Skin-Laminate-Design-and-Failure-Analysis
# Composite Wing Skin: Laminate Design and Failure Analysis

ANSYS ACP (Pre) + ANSYS Mechanical study of an 8-ply unidirectional carbon/epoxy wing skin, modelled as a root-clamped cantilever under a distributed load. Includes a laminate-theory hand check of the tip deflection and a ply failure assessment with the Composite Failure Tool using Hashin and Puck criteria.

**Status: work in progress.** The current layup does not pass the failure check at 250 N (see Results and Limitations).

## Model

| Item | Value |
|---|---|
| Geometry | Single aerofoil-shaped skin surface, loaded face area 51,058 mm² [CONFIRM: aerofoil section, chord and span] |
| Material | Epoxy Carbon UD (230 GPa) Prepreg, 0.15 mm per ply |
| Layup (bottom to top) | [0 / 25 / 0 / 25 / 25 / 0 / 25 / 0], 8 plies, 1.2 mm total |
| Layup type | Symmetric, unbalanced (B = 0; non-zero A16 and D16 give bend-twist and shear-extension coupling) |
| Mesh | 11,488 shell elements, 3 mm sizing |
| Support | Fixed support on the 2 root edges |
| Load | 250 N in -Y on the loaded face, ramped over 1 s (static structural) |
| Software | ANSYS 2026 R1 (student licence): ACP (Pre) for the layup, Mechanical for the solve |

### Ply table

| Ply | Angle | Thickness |
|---|---|---|
| P1L1 | 0° | 0.15 mm |
| P1L2 | 25° | 0.15 mm |
| P1L3 | 0° | 0.15 mm |
| P1L4 | 25° | 0.15 mm |
| P1L5 | 25° | 0.15 mm |
| P1L6 | 0° | 0.15 mm |
| P1L7 | 25° | 0.15 mm |
| P1L8 | 0° | 0.15 mm |

### ACP layup

![ACP thickness plot, 1.2 mm total](images/01-acp-thickness-1p2mm.png)

Ply definitions in Mechanical (imported from ACP):

![P1L1: UD, 0.15 mm, 0 degrees](images/02-ply-P1L1-0deg.png)
![P1L2: UD, 0.15 mm, 25 degrees](images/03-ply-P1L2-25deg.png)

### Boundary conditions

![Fixed support on 2 root edges](images/04-fixed-support.png)
![250 N force in -Y on the loaded face](images/05-force-250N.png)

## Results

| Result | Value |
|---|---|
| Tip deflection (Y), leading-edge side | 69.7 mm |
| Tip deflection (Y), trailing-edge side | 70.3 mm |
| Maximum total deformation | 70.3 mm |
| Maximum inverse reserve factor (IRF) | 3.13 |
| Mean IRF | 0.40 |

### Deformation

![Total deformation, maximum 70.3 mm at the tip](images/06-total-deformation.png)
![Tip deflection along the leading-edge side](images/07-tip-deflection-LE.png)
![Tip deflection along the trailing-edge side](images/08-tip-deflection-TE.png)

## Failure assessment

Ply failure is assessed with the ANSYS Composite Failure Tool. The inverse reserve factor (IRF) is the inverse of the load factor to failure: IRF above 1 means the ply has failed, and below 1 means it is safe (an IRF of 0.5 means the load could roughly double).

### Failure criteria used

- **Hashin:** treats fibre failure and matrix failure separately, with four modes: fibre tension, fibre compression, matrix tension and matrix compression. In its standard plane-stress form, failure is predicted when any index reaches 1. It tells you whether the fibres or the resin fail first.
- **Puck:** separates fibre failure from inter-fibre (matrix) failure and evaluates matrix failure on the fracture plane. It distinguishes three inter-fibre failure modes (A: transverse tension, B: moderate transverse compression, C: high transverse compression), which makes it better for judging delamination risk. It needs additional material constants beyond the basic stress limits [CONFIRM these were set in Engineering Data].
- **Maximum Stress:** simple per-direction check against the ply stress limits, used as a baseline [delete if not run].

Stress limits for Epoxy Carbon UD (230 GPa) Prepreg are taken from Engineering Data [CONFIRM values].

### Failure results

| Criterion | Max IRF | Critical ply | Critical mode |
|---|---|---|---|
| Maximum Stress | [x] | [ply] | [mode] |
| Hashin | [x] | [ply] | [mode] |
| Puck | [x] | [ply] | [mode] |

The failed region is concentrated near the root [CONFIRM from the plots].

![Inverse reserve factor, full skin, maximum 3.13](images/09-IRF-full.png)
![Inverse reserve factor, root region close-up](images/10-IRF-root.png)

## Hand check

Classical laminate theory gives a flexural modulus of about 96 GPa for this layup (using ply properties E1 = 121 GPa, E2 = 8.6 GPa, G12 = 4.7 GPa, nu12 = 0.27 [CONFIRM against Engineering Data]).

Treating the skin as a cantilever beam under a uniform load w = 250 N / 500 mm, with the second moment of area of the skin arc (about 640 mm^4), tip deflection = wL^4 / (8EI) = about 63 mm, which is within about 10% of the FE result (70.3 mm).

For comparison, a closed aerofoil shell would give about 8 mm, so the FE model is behaving as a single skin as intended.

## Limitations and next steps

- The layup fails at 250 N (maximum IRF 3.13 at the root). Next step: iterate the layup (more plies, thickness or angles) until the maximum IRF is below 1 with margin, then record the before and after.
- Von Mises stress is not a suitable measure for UD plies. Add fibre-direction and transverse ply stresses.
- No mesh convergence study yet.
- No physical test data.
- Student licence, so results are for learning and portfolio purposes only.
