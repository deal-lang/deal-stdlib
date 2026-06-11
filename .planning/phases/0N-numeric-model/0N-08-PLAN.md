---
track: 0N-numeric-model
lane: B
plan: 08
type: execute
wave: 1
depends_on: [09]          # header template only; runs parallel to 07
autonomous: false
files_modified:
  - packages/constants/codata.deal        # new
  - packages/constants/mod.deal           # new
  - packages/units/conversions.deal       # new
  - packages/units/imperial.deal          # completed
  - packages/units/mod.deal
  - deal.toml
requirements:
  - REQ-phase-N-numeric-model
source_adr: deal/.planning/decisions/ADR-deal-stdlib-numeric-model.md
must_haves:
  truths:
    - "A deal.std.constants package ships CODATA 2022 fundamental constants as DEAL attribute defs"
    - "Imperial coverage is completed against NIST SP 811 with explicit conversion call forms for all sourced domains"
    - "SL-7 policy is documented: SI is canonical and always available; imperial is opt-in via explicit import"
    - "constants/*.deal and units/*.deal parse under deal parse with exit 0"
  artifacts:
    - path: "packages/constants/codata.deal"
      provides: "Fundamental physical constants (c, h, hbar, k_B, N_A, e, G, m_e, ...) — CODATA 2022"
      contains: "package deal.std.constants"
    - path: "packages/units/conversions.deal"
      provides: "Explicit to_<SI>() converter forms across all unit domains (extends D-57 set)"
---

<objective>
Ship the public-domain content: the CODATA 2022 fundamental constants package and the completed
NIST SP 811 conversion layer. Document SL-7 (SI canonical, imperial opt-in). No compiler change;
authoring + parse verification only. Constants carry declared precision where the ADR's
annotation syntax exists — but since that syntax is Lane A (N-01), encode the *value* now and
leave a `// precision: <CODATA stated uncertainty>` doc comment to be upgraded to `sig`/`±`
annotations once N-01 lands.

Output: `packages/constants/{codata.deal, mod.deal}`, `packages/units/conversions.deal`,
completed `imperial.deal`, updated `mod.deal` + `deal.toml`.
</objective>

<context>
@deal-stdlib/packages/units/imperial.deal     # current imperial subset + D-57 converters
@deal-stdlib/deal.toml                          # [workspace] packages glob
Sources: NIST CODATA 2022 (https://physics.nist.gov/cuu/Constants/), NIST SP 811 (2008).
Both public domain. Apply the N-09 NOTICE header to both new files.
</context>

<task id="08-1" name="Create constants package + register it">
Create `packages/constants/` with `mod.deal` (barrel) and `codata.deal`. In `codata.deal`,
`package deal.std.constants;` and import the dimensions it needs from `deal.std.units`. Encode
each constant as an `attribute def` specializing its dimension (or `Dimensionless`), with the
CODATA value in `si_factor` form OR as a `derived attribute` holding the value — pick whichever
parses cleanly under the locked grammar (probe with `deal parse` first; the existing unit-def
pattern is the safe default). Minimum CODATA 2022 set:

| Const | Symbol | Value (SI) | Dimension |
|-------|--------|-----------|-----------|
| speed of light | c | 299792458 (exact) | Speed |
| Planck | h | 6.62607015e-34 (exact) | (J·s) — Action |
| reduced Planck | hbar | 1.054571817e-34 | Action |
| Boltzmann | k_B | 1.380649e-23 (exact) | (J/K) HeatCapacity |
| Avogadro | N_A | 6.02214076e23 (exact) | (1/mol) |
| elementary charge | e | 1.602176634e-19 (exact) | Charge |
| electron mass | m_e | 9.1093837015e-31 | Mass |
| proton mass | m_p | 1.67262192369e-27 | Mass |
| gravitational | G | 6.67430e-11 | (m³/(kg·s²)) |
| std gravity | g_0 | 9.80665 (exact) | Acceleration |
| vacuum permittivity | eps_0 | 8.8541878128e-12 | Permittivity |
| vacuum permeability | mu_0 | 1.25663706212e-6 | Permeability |
| gas constant | R | 8.314462618 | (J/(mol·K)) MolarEnergy/TH |
| Stefan-Boltzmann | sigma | 5.670374419e-8 | (W/(m²·K⁴)) |

Add `Action` (J·s = [1,2,-1,0,0,0,0]) to `dimensions.deal` if N-07 hasn't (coordinate to avoid
a dup def). For each non-exact constant add `// precision: <CODATA stated rel. uncertainty>`
(upgrade to `± e-N` once N-01 ships). Register the package: add `"packages/*"` already covers it
via the glob — confirm `deal.toml [workspace] packages = ["packages/*"]` picks up `constants/`;
if package resolution needs an explicit entry, add it.
</task>

<task id="08-2" name="Complete imperial + conversions">
Extend `imperial.deal` to the full SP 811 set the showcase + common defense/aero work needs,
beyond today's subset. Add at least: `psi`(Pressure, 6894.76 Pa), `ksi`, `inHg`, `atm`(101325),
`gal_us`(Volume, 3.78541e-3 m³), `qt`, `pt`, `floz`, `nmi`(Length, 1852), `acre`(Area, 4046.86),
`ftlb`(Energy/Torque, 1.35582 — note the Torque/Energy ambiguity in a doc comment), `degR`
(Temperature, 5/9). Move the `to_<SI>()` converter forms out of `imperial.deal` into a new
`packages/units/conversions.deal` and add the missing-domain converters: `to_Pa`, `to_m2`,
`to_m3`, `to_Hz`, `to_F_cap`, plus keep the existing `to_kg/to_m/to_s/to_N/to_J/to_W/to_K/to_m_per_s`.
Each converter is `attribute def to_X <<specializes>> Dim { si_factor = 1.0; }` per D-57.
</task>

<task id="08-3" name="Document SL-7 + update barrel">
At the top of `conversions.deal` and in `README.md`, document SL-7: SI units are canonical and
(per SL-3, assumed) always available; imperial units and converters require an explicit
`import deal.std.units.{lb, ft, to_kg};`. Update `packages/units/mod.deal` to export the new
imperial units and the relocated converter forms from `conversions.deal` (not `imperial.deal`).
Create `packages/constants/mod.deal` exporting all constants.
</task>

<verification>
<automated>cd /Users/dunnock/projects/deal-lang/deal && for f in ../deal-stdlib/packages/units/*.deal ../deal-stdlib/packages/constants/*.deal; do cargo run -q -p deal -- parse "$f" >/dev/null || { echo "PARSE FAIL: $f"; exit 1; }; done && echo STDLIB_PARSE_OK</automated>
- All `units/` + `constants/` files parse, exit 0.
- Probe: `import deal.std.constants.{c, k_B}; import deal.std.units.{psi, to_Pa};`
  `attribute speed : Speed = c;` and `to_Pa(psi(100))` — `deal check` exits 0.
- `c`, `k_B`, `N_A`, `e`, `g_0` carry exact values; non-exact constants carry a `precision:` note.
- SL-7 documented; imperial requires explicit import (SI-only model needs no imperial import).
</verification>

<acceptance>
`deal.std.constants` ships and parses; SP 811 coverage completed with converters for all sourced
domains; SL-7 documented. Feeds roadmap N-11 exit gate and unblocks N-08's `precision:` notes for
upgrade once Lane A N-01 lands.
</acceptance>
