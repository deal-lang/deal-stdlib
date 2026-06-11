---
track: 0N-numeric-model
lane: B
plan: 07
type: execute
wave: 1
depends_on: [09]          # for the header template only; not a logic dependency
autonomous: false
files_modified:
  - packages/units/dimensions.deal
  - packages/units/dimensionless.deal     # new
  - packages/units/si_catalog.deal        # new
  - packages/units/si.deal                # percent removed
  - packages/units/mod.deal
requirements:
  - REQ-phase-N-numeric-model
source_adr: deal/.planning/decisions/ADR-deal-stdlib-numeric-model.md
adr_confirmation: "#1 (content half)"
must_haves:
  truths:
    - "deal-stdlib declares the ISO 80000 derived quantities as dimension defs carrying correct 7-exponent vectors"
    - "A Dimensionless dimension exists and percent specializes it (no longer the Mass hack)"
    - "Same-vector named quantities (Torque/Energy, Frequency/AngularVelocity, etc.) are each declared distinctly and flagged for Lane A enforcement"
    - "Each Modelica-derived file carries the NOTICE header (from N-09)"
    - "Every units/*.deal parses under deal parse with exit 0"
  artifacts:
    - path: "packages/units/dimensions.deal"
      provides: "Expanded SI base + derived dimension catalog (~80 quantities)"
      contains: "attribute def Pressure"
    - path: "packages/units/dimensionless.deal"
      provides: "Dimensionless, Angle, SolidAngle (all-zero vectors, named-distinct)"
    - path: "packages/units/si_catalog.deal"
      provides: "Unit constructors for the new derived quantities (Pa, Hz, F, H, T, Wb, S, ...)"
  key_links:
    - from: "packages/units/si_catalog.deal"
      to: "packages/units/dimensions.deal"
      via: "unit defs <<specializes>> derived dimensions"
      pattern: "Pressure|Frequency|Capacitance"
---

<objective>
Grow the quantity catalog from the current 15 dimensions / ~40 units to ISO 80000 coverage,
using the locked Option A encoding and the existing vector algebra (no compiler change). Add
the missing Dimensionless dimension and re-home `percent`. Declare same-vector quantities as
distinct named dimensions, flagged for the later Lane A enforcement pass. This satisfies the
content half of ADR Confirmation #1.

Output: expanded `dimensions.deal`, new `dimensionless.deal` and `si_catalog.deal`, `percent`
removed from `si.deal`, `mod.deal` barrel updated.
</objective>

<context>
@deal/.planning/decisions/ADR-phase-4-dimension-metadata-syntax.md   # encoding contract
@deal-stdlib/packages/units/dimensions.deal                          # current 15 dims
@deal-stdlib/packages/units/si.deal                                  # current units + percent hack
Vector order is [si_M, si_L, si_T, si_I, si_TH, si_N, si_J]. Authoritative source for the full
270+ enumeration: Modelica.Units.SI (MSL). The table below is the verified core set to author
first; extend to full ISO 80000 coverage from MSL using the same method.
</context>

<task id="07-1" name="Add Dimensionless + angle quantities">
Create `packages/units/dimensionless.deal` (package `deal.std.units`, NOTICE header). Declare,
each with ALL-ZERO exponent vectors but kept as distinct named defs:

| Dimension     | Note |
|---------------|------|
| `Dimensionless` | base dimensionless quantity (ratios, counts, efficiency) |
| `Angle`         | radian — dimensionless, named-distinct (@same_vector_note: Dimensionless) |
| `SolidAngle`    | steradian — dimensionless, named-distinct |
| `Strain`        | dimensionless ratio |

Then in `si.deal`, **delete** the `percent <<specializes>> Mass` block and re-declare it in
`dimensionless.deal` as `percent <<specializes>> Dimensionless { si_factor = 0.01; }`. Add
unit defs `rad <<specializes>> Angle`, `deg <<specializes>> Angle { si_factor = 0.0174533; }`,
`sr <<specializes>> SolidAngle`.
</task>

<task id="07-2" name="Expand dimensions.deal — derived quantities">
Append the following derived dimension defs to `dimensions.deal` (add the NOTICE header at top).
Verified 7-exponent vectors [M, L, T, I, TH, N, J]:

| Dimension                | M | L | T | I | TH| N | J | SI unit | same-vector flag |
|--------------------------|---|---|---|---|---|---|---|---------|------------------|
| Area                     | 0 | 2 | 0 | 0 | 0 | 0 | 0 | m²      |                  |
| Volume                   | 0 | 3 | 0 | 0 | 0 | 0 | 0 | m³      |                  |
| Acceleration             | 0 | 1 |-2 | 0 | 0 | 0 | 0 | m/s²    |                  |
| AngularVelocity          | 0 | 0 |-1 | 0 | 0 | 0 | 0 | rad/s   | =Frequency       |
| AngularAcceleration      | 0 | 0 |-2 | 0 | 0 | 0 | 0 | rad/s²  |                  |
| Frequency                | 0 | 0 |-1 | 0 | 0 | 0 | 0 | Hz      | =AngularVelocity |
| Wavenumber               | 0 |-1 | 0 | 0 | 0 | 0 | 0 | 1/m     |                  |
| Density                  | 1 |-3 | 0 | 0 | 0 | 0 | 0 | kg/m³   |                  |
| SpecificVolume           |-1 | 3 | 0 | 0 | 0 | 0 | 0 | m³/kg   |                  |
| Momentum                 | 1 | 1 |-1 | 0 | 0 | 0 | 0 | kg·m/s  |                  |
| Pressure                 | 1 |-1 |-2 | 0 | 0 | 0 | 0 | Pa      | =Stress          |
| Stress                   | 1 |-1 |-2 | 0 | 0 | 0 | 0 | Pa      | =Pressure        |
| Torque                   | 1 | 2 |-2 | 0 | 0 | 0 | 0 | N·m     | =Energy          |
| DynamicViscosity         | 1 |-1 |-1 | 0 | 0 | 0 | 0 | Pa·s    |                  |
| KinematicViscosity       | 0 | 2 |-1 | 0 | 0 | 0 | 0 | m²/s    |                  |
| SurfaceTension           | 1 | 0 |-2 | 0 | 0 | 0 | 0 | N/m     |                  |
| MomentOfInertia          | 1 | 2 | 0 | 0 | 0 | 0 | 0 | kg·m²   |                  |
| HeatCapacity / Entropy   | 1 | 2 |-2 | 0 |-1 | 0 | 0 | J/K     |                  |
| SpecificHeatCapacity     | 0 | 2 |-2 | 0 |-1 | 0 | 0 | J/(kg·K)|                  |
| ThermalConductivity      | 1 | 1 |-3 | 0 |-1 | 0 | 0 | W/(m·K) |                  |
| HeatFlux                 | 1 | 0 |-3 | 0 | 0 | 0 | 0 | W/m²    |                  |
| Capacitance              |-1 |-2 | 4 | 2 | 0 | 0 | 0 | F       |                  |
| Conductance              |-1 |-2 | 3 | 2 | 0 | 0 | 0 | S       |                  |
| MagneticFlux             | 1 | 2 |-2 |-1 | 0 | 0 | 0 | Wb      |                  |
| MagneticFluxDensity      | 1 | 0 |-2 |-1 | 0 | 0 | 0 | T       |                  |
| Inductance               | 1 | 2 |-2 |-2 | 0 | 0 | 0 | H       |                  |
| ElectricFieldStrength    | 1 | 1 |-3 |-1 | 0 | 0 | 0 | V/m     |                  |
| Permittivity             |-1 |-3 | 4 | 2 | 0 | 0 | 0 | F/m     |                  |
| Permeability             | 1 | 1 |-2 |-2 | 0 | 0 | 0 | H/m     |                  |
| LuminousFlux             | 0 | 0 | 0 | 0 | 0 | 0 | 1 | lm      |                  |
| Illuminance              | 0 |-2 | 0 | 0 | 0 | 0 | 1 | lx      |                  |
| MolarMass                | 1 | 0 | 0 | 0 | 0 |-1 | 0 | kg/mol  |                  |
| MolarConcentration       | 0 |-3 | 0 | 0 | 0 | 1 | 0 | mol/m³  |                  |
| CatalyticActivity        | 0 | 0 |-1 | 0 | 0 | 1 | 0 | kat     | =Frequency·N     |
| AbsorbedDose / DoseEquiv | 0 | 2 |-2 | 0 | 0 | 0 | 0 | Gy / Sv |                  |
| Radioactivity            | 0 | 0 |-1 | 0 | 0 | 0 | 0 | Bq      | =Frequency       |

For every row whose "same-vector flag" is non-empty, add the doc comment
`@same_vector_note: distinct-from <Name> — Lane A enforcement (N-00 ③)`.

Then extend toward full ISO 80000 coverage by mechanically translating the remaining
`Modelica.Units.SI` type names with the same encoding (target ≈ 80 dimensions; ADR
Confirmation #1 counts the full type set including units below).
</task>

<task id="07-3" name="Author si_catalog.deal — unit constructors">
Create `packages/units/si_catalog.deal` (NOTICE header). For each new derived dimension add the
named SI unit `<<specializes>>` it with `si_factor = 1.0`, plus the standard decimal-prefix
constructors. Prefix policy: ship the prefixes engineers actually use per quantity (don't
mechanically cross every prefix with every unit). Minimum set:

| Quantity     | Constructors |
|--------------|-------------|
| Pressure     | `Pa`, `kPa`, `MPa`, `GPa`, `bar`(=1e5), `mbar` |
| Frequency    | `Hz`, `kHz`, `MHz`, `GHz` |
| Capacitance  | `F`, `mF`, `uF`, `nF`, `pF` |
| Inductance   | `H`, `mH`, `uH`, `nH` |
| MagFluxDensity| `T`, `mT`, `uT`, `gauss`(=1e-4) |
| MagneticFlux | `Wb` |
| Conductance  | `S`, `mS`, `uS` |
| Area         | `m2`, `cm2`, `mm2`, `km2` |
| Volume       | `m3`, `L`(=1e-3), `mL`, `cm3` |
| Acceleration | `m_per_s2`, `g_accel`(=9.80665) |
| AngularVel.  | `rad_per_s`, `rpm`(=0.1047198) |
| Density      | `kg_per_m3`, `g_per_cm3`(=1000) |
| MolarConc.   | `mol_per_m3`, `mol_per_L`(=1000) |
| CatalyticAct.| `kat` |
| Dose         | `Gy`, `Sv`, `mGy`, `mSv` |
| Radioactivity| `Bq`, `kBq`, `MBq`, `GBq`, `Ci`(=3.7e10) |
| Illuminance  | `lx` |
| LuminousFlux | `lm` |

Use the same `attribute def X <<specializes>> Dim { attribute si_factor : Real = F; }` form as
`si.deal`. Keep one ── section header ── comment per quantity, matching the existing style.
</task>

<task id="07-4" name="Update barrel export (mod.deal)">
Extend `packages/units/mod.deal`:
- `export dimensionless.{Dimensionless, Angle, SolidAngle, Strain, percent, rad, deg, sr};`
- `export dimensions.{Area, Volume, Acceleration, Frequency, Pressure, Stress, Torque, ...};` (all new)
- `export si_catalog.{Pa, kPa, MPa, Hz, kHz, F, uF, H, T, S, ...};` (all new constructors)
Remove `percent` from the `export si.{…}` line (it moved to `dimensionless`).
</task>

<verification>
<automated>cd /Users/dunnock/projects/deal-lang/deal && for f in ../deal-stdlib/packages/units/*.deal; do cargo run -q -p deal -- parse "$f" >/dev/null || { echo "PARSE FAIL: $f"; exit 1; }; done && echo UNITS_PARSE_OK</automated>
- All `units/*.deal` parse, exit 0 (`UNITS_PARSE_OK`).
- Write a probe consumer `.deal`: `import deal.std.units.{kPa, Hz, uF};` with
  `attribute p : Pressure = kPa(101); attribute clk : Frequency = MHz(50);` — `deal check`
  exits 0. Then `attribute bad : Pressure = Hz(1);` must emit `error[E2500]`.
- Confirm `percent` resolves as `Dimensionless` (grep the def; no `<<specializes>> Mass`).
- Confirm each same-vector dimension carries a `@same_vector_note`.
</verification>

<acceptance>
ADR Confirmation #1 (content half): the ISO 80000 quantity catalog is declared, parses, and
checks; same-vector names are distinct and flagged; `percent` is Dimensionless; sourced files
carry the NOTICE header. Full 270+ count completes by extending §07-2/§07-3 from MSL.
</acceptance>

<notes>
- The exponent vectors in §07-2 are the authoring source of truth — they are physics-verified.
  Cross-check any MSL-extended additions against ISO 80000-1 before committing.
- Do NOT attempt Torque≠Energy *enforcement* here; declaring them distinct + flagging is the
  whole Lane B obligation. Enforcement is Lane A (sema, gated on N-00 ③).
- `bar`, `gauss`, `Ci`, `rpm`, `g_accel` are convenience non-SI constructors with explicit
  factors — acceptable here; SL-7 (imperial opt-in) is handled in N-08.
</notes>
