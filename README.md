# Unidirectional (UD) Carbon Layup: Composite Wing Skin Laminate Design and Failure Analysis

ANSYS ACP (Pre) + ANSYS Mechanical study of an 8-ply unidirectional carbon/epoxy wing skin, modelled as a root-clamped cantilever under a distributed load. Includes a classical laminate theory (CLT) hand check of the tip deflection, kinematic tip-twist calculation, and ply failure assessment with the Composite Failure Tool using **Tsai-Wu**, **Hashin**, and **Puck** criteria.

> **Note on Project Images:** All ANSYS ACP layup, mesh, boundary condition, deformation, and Composite Failure Tool screenshots are uploaded in the repository above.

**Status:** Work in progress. The current 8-ply layup does not pass the failure check at 250 N (see *Results* and *Limitations*).

---

## 1. Model Setup

| Item | Value |
| :--- | :--- |
| **Geometry** | Single aerofoil-shaped skin surface (NACA 2412 upper skin, 100 mm chord, 500 mm span; loaded face area 51,058 mm²) |
| **Material** | Epoxy Carbon UD (230 GPa) Prepreg, 0.15 mm per ply |
| **Layup (bottom to top)** | `[0 / 25 / 0 / 25 / 25 / 0 / 25 / 0]` (`[0 / 25 / 0 / 25]s`), 8 plies, 1.2 mm total |
| **Layup Type** | Symmetric, unbalanced (`B = 0`; non-zero `A16` and `D16` produce bend-twist and shear-extension coupling) |
| **Mesh** | 11,488 shell elements, 3 mm sizing |
| **Support** | Fixed support on the root edges (`redge(ACP (Pre))`) |
| **Load** | 250 N in `-Y` on the loaded skin (`skin(ACP (Pre))`), ramped over 1 s (Static Structural) |
| **Software** | ANSYS 2026 R1 (Student Licence): ACP (Pre) for layup definition, Mechanical for structural solve |

### Ply Stackup Table

| Ply | Angle | Thickness |
| :--- | :---: | :---: |
| `P1L1` | 0° | 0.15 mm |
| `P1L2` | 25° | 0.15 mm |
| `P1L3` | 0° | 0.15 mm |
| `P1L4` | 25° | 0.15 mm |
| `P1L5` | 25° | 0.15 mm |
| `P1L6` | 0° | 0.15 mm |
| `P1L7` | 25° | 0.15 mm |
| `P1L8` | 0° | 0.15 mm |

---

## 2. Results

| Result | Value |
| :--- | :--- |
| **Tip deflection (`Y`), leading-edge side** | 69.7 mm (`69.694 mm`) |
| **Tip deflection (`Y`), trailing-edge side** | 70.3 mm (`70.317 mm`) |
| **Tip twist angle ($\Delta\theta$)** | 0.36° (`atan((70.317 - 69.694) / 100) = 0.357°`) |
| **Maximum total deformation** | 70.3 mm |
| **Maximum inverse reserve factor (IRF — Tsai-Wu baseline)** | 3.13 |
| **Mean IRF (Tsai-Wu baseline)** | 0.40 |

---

## 3. Failure Assessment

Ply failure is evaluated in the ANSYS Composite Failure Tool using **Inverse Reserve Factor (`IRF = 1 / RF`)**:
* **`IRF > 1.0`**: Ply failure predicted.
* **`IRF < 1.0`**: Ply survives (`1 / IRF` gives the load factor to failure).

### Failure Criteria Configured (2D Plane-Stress)
* **Tsai-Wu (`tw`, Weighting Factor = 1):** Quadratic stress-interaction failure envelope combining longitudinal, transverse, and in-plane shear stresses into a single scalar index.
* **Hashin (`hf`, `hm`, Weighting Factor = 1):** Separates fibre failure (`hf`) and matrix failure (`hm`) into distinct plane-stress modes (fibre tension/compression and matrix tension/compression), identifying whether the carbon fibres or epoxy resin fail first.
* **Puck (`pf`, `pmA`, `pmB`, `pmC`, Weighting Factor = 1):** Separates fibre failure (`pf`) from inter-fibre matrix failure evaluated on the physical fracture plane through the ply thickness:
  * **Mode A (`pmA`):** Transverse tension combined with shear ($0^\circ$ fracture plane).
  * **Mode B (`pmB`):** Moderate transverse compression, increasing shear resistance via internal friction.
  * **Mode C (`pmC`):** High transverse compression causing an inclined wedge fracture ($\approx 53^\circ$), indicating delamination risk.
  * **ANSYS Puck Constants Applied:** `p21(+) = 0.325`, `p21(-) = 0.275`, `p22(+) = 0.225`, `p22(-) = 0.225` (with degradation factors `s = 0.5`, `M = 0.5`, and interface weakening factor `0.8`).

### Failure Results Summary (Epoxy Carbon UD 230 GPa Prepreg)
Material stress limits from ANSYS Engineering Data: $X_t = 2231\text{ MPa}$, $X_c = 1082\text{ MPa}$, $Y_t = 29\text{ MPa}$, $Y_c = 100\text{ MPa}$, $S_{12} = 60\text{ MPa}$.

| Evaluation Run | Active Criteria | Max IRF | Critical Ply | Governing Mechanism |
| :--- | :--- | :---: | :---: | :--- |
| **Baseline Envelope** | Tsai-Wu (`tw`) | 3.13 | `25°` plies (root) | Combined transverse tension & in-plane shear |
| **Multi-Criteria Envelope** | Tsai-Wu (`tw`) + Hashin (`hf`, `hm`) + Puck (`pf`, `pmA`, `pmB`, `pmC`) | `[x]` | `[x]` | Matrix tension / inter-fibre shear (`hm` / `pmA`) at root |

The failed region (`IRF > 1.0`) is concentrated near the clamped root boundary (`redge`), where bending moment and restrained Poisson/shear-extension coupling peak.

---

## 4. Hand Check (Classical Laminate Theory)

Run `python hand_calc/clt_check.py` (requires `numpy`) to reproduce the analytical checks:

* **Laminate Stiffness:** Using ANSYS `Epoxy Carbon UD (230 GPa) Prepreg` properties ($E_1 = 121\text{ GPa}$, $E_2 = 8.6\text{ GPa}$, $G_{12} = 4.7\text{ GPa}$, $\nu_{12} = 0.27$), classical laminate theory gives an in-plane modulus $E_x = 77.6\text{ GPa}$, flexural modulus $E_{x,\text{flex}} = 96.2\text{ GPa}$, shear modulus $G_{xy} = 9.0\text{ GPa}$, and bend-twist coupling ratio $D_{16}/D_{11} = 0.098$.
* **Tip Deflection:** Modelling the single upper skin as a cantilever beam under a uniform load $w = 250\text{ N} / 500\text{ mm}$ with skin-arc second moment of area $I \approx 640\text{ mm}^4$ yields:
  $$\delta_{\text{tip}} = \frac{w L^4}{8 E I} \approx 78.6\text{ mm (membrane } E_x\text{) to } 63.4\text{ mm (flexural } E_{x,\text{flex}}\text{)}$$
  This brackets the ANSYS FEA deflection of **70.3 mm** within ~11% (whereas a closed NACA 2412 aerofoil shell would deflect ~10 mm, confirming the model behaves as a single open skin).
* **Root Ply Check:** Beam-bending strain transformation predicts `IRF > 1.0` in the `25°` plies (`Hashin Matrix Tension = 1.24`, `Tsai-Wu = 1.24`, `Max Stress = 1.01`) while `0°` and `25°` fibre indices remain below `1.0`, confirming resin/matrix cracking rather than fibre breakage.

---

## 5. Limitations and Next Steps

* The current 8-ply UD layup fails at 250 N (`Max IRF = 3.13` at the root). Next step: add off-axis/woven reinforcement or increase laminate thickness until `Max IRF < 1.0` with design margin, and compare against a closed-shell `2x2 Twill` woven laminate.
* Von Mises stress is not used as a failure criterion because it is invalid for orthotropic composite plies; evaluation relies on ply-coordinate stresses and Tsai-Wu, Hashin, and Puck criteria.
* No mesh convergence study or physical coupon test data is included yet.
* Built under the ANSYS Student Licence for engineering learning and portfolio demonstration.only.
* """
Classical Laminate Theory (CLT), tip-deflection, tip-twist, and root ply failure
hand-check script for the 8-ply Unidirectional (UD) NACA 2412 composite wing skin.

Units: mm, N, MPa.
"""
import numpy as np

# ---------------- ANSYS Engineering Data Inputs ----------------
UD = dict(E1=121000., E2=8600., G12=4700., nu12=0.27)
UD_STRENGTH = dict(Xt=2231., Xc=1082., Yt=29., Yc=100., S12=60., S23=32.)

ANGLES = [0, 25, 0, 25, 25, 0, 25, 0]   # [0 / 25 / 0 / 25]s
PLY_THICKNESS = 0.15                     # mm (total h = 1.20 mm)

CHORD, M_CAMBER, P_CAMBER, T_RATIO = 100., 0.02, 0.4, 0.12  # NACA 2412
SPAN, LOAD = 500., 250.                                      # mm, N (uniform over span)
# ---------------------------------------------------------------


def q_matrix(E1, E2, G12, nu12):
    nu21 = nu12 * E2 / E1
    d = 1 - nu12 * nu21
    return np.array([[E1 / d, nu12 * E2 / d, 0], [nu12 * E2 / d, E2 / d, 0], [0, 0, G12]])


def t_matrix(theta):
    c, s = np.cos(np.radians(theta)), np.sin(np.radians(theta))
    return np.array([[c * c, s * s, 2 * c * s], [s * s, c * c, -2 * c * s], [-c * s, c * s, c * c - s * s]])


R = np.diag([1, 1, 2])


def q_bar(Q, theta):
    T = t_matrix(theta)
    return np.linalg.inv(T) @ Q @ R @ T @ np.linalg.inv(R)


def clt(ply, angles, t):
    Q = q_matrix(**ply)
    n, h = len(angles), len(angles) * t
    z = np.linspace(-h / 2, h / 2, n + 1)
    A = sum(q_bar(Q, a) * (z[k + 1] - z[k]) for k, a in enumerate(angles))
    B = sum(q_bar(Q, a) * (z[k + 1] ** 2 - z[k] ** 2) / 2 for k, a in enumerate(angles))
    D = sum(q_bar(Q, a) * (z[k + 1] ** 3 - z[k] ** 3) / 3 for k, a in enumerate(angles))
    a_inv, d_inv = np.linalg.inv(A), np.linalg.inv(D)
    return dict(
        h=h, A=A, B=B, D=D,
        Ex_mem=1 / (a_inv[0, 0] * h),
        Gxy_mem=1 / (a_inv[2, 2] * h),
        Ex_flex=12 / (d_inv[0, 0] * h ** 3),
        D16_D11=D[0, 2] / D[0, 0],
        A16_A11=A[0, 2] / A[0, 0],
    )


def naca_section(c=CHORD, m=M_CAMBER, p=P_CAMBER, tt=T_RATIO):
    x = np.linspace(0, 1, 4001)
    yt = 5 * tt * (0.2969 * np.sqrt(x) - 0.1260 * x - 0.3516 * x**2 + 0.2843 * x**3 - 0.1015 * x**4)
    yc = np.where(x < p, m / p**2 * (2 * p * x - x**2), m / (1 - p)**2 * ((1 - 2 * p) + 2 * p * x - x**2))
    dyc = np.where(x < p, 2 * m / p**2 * (p - x), 2 * m / (1 - p)**2 * (p - x))
    th = np.arctan(dyc)
    xu, yu = (x - yt * np.sin(th)) * c, (yc + yt * np.cos(th)) * c
    xl, yl = (x + yt * np.sin(th)) * c, (yc - yt * np.cos(th)) * c
    return xu, yu, xl, yl


def second_moment(xs, ys):
    """Second moment of area per unit wall thickness about section centroid."""
    ds = np.hypot(np.diff(xs), np.diff(ys))
    ym = (ys[1:] + ys[:-1]) / 2
    yb = np.sum(ym * ds) / np.sum(ds)
    return np.sum((ym - yb) ** 2 * ds), (ys - yb).max(), (ys - yb).min()


def root_ply_check(ply, angles, t, strength, I_per_t, y_ext):
    """Beam-bending root ply stresses and linear Inverse Reserve Factors (IRF = 1 / RF)."""
    props = clt(ply, angles, t)
    h = props["h"]
    M_root = LOAD * SPAN / 2
    kappa = M_root / (props["Ex_mem"] * I_per_t * h)
    Q = q_matrix(**ply)
    out = []
    for label, y in (("tension side", y_ext[0]), ("compression side", y_ext[1])):
        eps_x = kappa * y
        N = np.array([props["Ex_mem"] * h * eps_x, 0, 0])
        e_lam = np.linalg.inv(props["A"]) @ N
        for ang in sorted(set(angles)):
            e = t_matrix(ang) @ np.array([e_lam[0], e_lam[1], e_lam[2] / 2])
            s1, s2, t12 = Q @ np.array([e[0], e[1], 2 * e[2]])
            S = strength
            max_stress = max(
                abs(s1) / (S["Xt"] if s1 >= 0 else S["Xc"]),
                abs(s2) / (S["Yt"] if s2 >= 0 else S["Yc"]),
                abs(t12) / S["S12"],
            )
            hashin_ft = np.sqrt((s1 / S["Xt"])**2 + (t12 / S["S12"])**2) if s1 >= 0 else 0.0
            hashin_fc = abs(s1) / S["Xc"] if s1 < 0 else 0.0
            hashin_mt = np.sqrt((s2 / S["Yt"])**2 + (t12 / S["S12"])**2) if s2 >= 0 else 0.0
            hashin_mc = np.sqrt(max(
                0.0,
                (s2 / (2 * S["S23"]))**2 + ((S["Yc"] / (2 * S["S23"]))**2 - 1) * (s2 / S["Yc"]) + (t12 / S["S12"])**2
            )) if s2 < 0 else 0.0
            F11, F22, F66 = 1 / (S["Xt"] * S["Xc"]), 1 / (S["Yt"] * S["Yc"]), 1 / S["S12"]**2
            F1, F2 = 1 / S["Xt"] - 1 / S["Xc"], 1 / S["Yt"] - 1 / S["Yc"]
            F12 = -0.5 * np.sqrt(F11 * F22)
            fi_tw = F1 * s1 + F2 * s2 + F11 * s1**2 + F22 * s2**2 + F66 * t12**2 + 2 * F12 * s1 * s2
            tsai_wu = np.sqrt(max(0.0, fi_tw))
            out.append((label, ang, s1, s2, t12, max_stress, hashin_ft, hashin_fc, hashin_mt, hashin_mc, tsai_wu))
    return out


def twist_deg(dy_te, dy_le, chord=CHORD):
    """Tip twist angle (degrees) from trailing-edge and leading-edge vertical displacements."""
    return np.degrees(np.arctan((abs(dy_te) - abs(dy_le)) / chord))


if __name__ == "__main__":
    p = clt(UD, ANGLES, PLY_THICKNESS)
    print(f"== CLT: UD [0/25/0/25]s (h = {p['h']:.2f} mm) ==")
    print(f"Ex_mem = {p['Ex_mem']:.0f} MPa | Ex_flex = {p['Ex_flex']:.0f} MPa | Gxy = {p['Gxy_mem']:.0f} MPa")
    print(f"D16/D11 = {p['D16_D11']:.3f} | A16/A11 = {p['A16_A11']:.3f} | max(|B|) = {abs(p['B']).max():.2e}")

    xu, yu, xl, yl = naca_section()
    I_open, ymax, ymin = second_moment(xu, yu)
    I_closed, _, _ = second_moment(np.concatenate([xu, xl[::-1]]), np.concatenate([yu, yl[::-1]]))
    w = LOAD / SPAN
    print("\n== Cantilever Tip Deflection Check ==")
    print(f"Open skin (Ex_mem):  {w * SPAN**4 / (8 * p['Ex_mem'] * I_open * p['h']):6.1f} mm")
    print(f"Open skin (Ex_flex): {w * SPAN**4 / (8 * p['Ex_flex'] * I_open * p['h']):6.1f} mm")
    print(f"Closed shell check:  {w * SPAN**4 / (8 * p['Ex_mem'] * I_closed * p['h']):6.1f} mm")
    print("ANSYS FEA result:      70.3 mm")

    print("\n== Root Ply Failure Check (Linear IRF) ==")
    print("side, ply, s1, s2, t12, MaxStress, HashinFT, HashinFC, HashinMT, HashinMC, TsaiWu")
    for row in root_ply_check(UD, ANGLES, PLY_THICKNESS, UD_STRENGTH, I_open, (-ymin, -ymax)):
        print(row[0], row[1], *[f"{v:.2f}" for v in row[2:]])

    print(f"\n== Kinematic Tip Twist ==\n{twist_deg(70.317, 69.694):.3f} deg")
