---
track: 0N-numeric-model
lane: B
plan: "07"
subsystem: stdlib
tags: [deal-stdlib, iso-80000, dimension-catalog, dimensionless, same-vector, parse-verified]

# Dependency graph
requires:
  - plan: 0N-09
    provides: NOTICE.md + HEADER-TEMPLATE.txt (attribution header pasted into sourced files)
provides:
  - deal-stdlib dimensions.deal expanded 15 -> 58 dimensions (ISO 80000 derived quantities)
  - deal-stdlib dimensionless.deal (Dimensionless, Angle, SolidAngle, Strain + percent re-homed)
  - deal-stdlib si_catalog.deal (120 derived-quantity unit constructors)
  - same-vector quantities declared distinctly + tagged @same_vector_note for Lane A enforcement
affects: [0N-08, 0N-11-integration, Lane-A-enforcement (N-00 ③)]

# Tech tracking
patterns-established:
  - "Derived dimension: attribute def Name { si_M..si_J (x7) } with physics-verified [M,L,T,I,TH,N,J]"
  - "Same-vector distinction: distinct named defs + @same_vector_note doc comment as the Lane A enforcement list"
  - "Dimensionless family (all-zero vector) kept as distinct named defs (Dimensionless/Angle/SolidAngle/Strain)"
  - "Prefix policy: ship the prefixes used per quantity, not the full prefix x unit cross-product"

key-files:
  created:
    - packages/units/dimensionless.deal
    - packages/units/si_catalog.deal
  modified:
    - packages/units/dimensions.deal   # +38 derived dims, Energy @same_vector_note, NOTICE header
    - packages/units/si.deal           # percent block removed (moved to dimensionless.deal)
    - packages/units/mod.deal          # barrel: new dims + units; percent now from dimensionless

key-decisions:
  - "Split N-07: declaring same-vector names is content (done here); ENFORCING Torque≠Energy is Lane A sema, gated on N-00 ③ — reclassified out of Lane B"
  - "No dimension-equation table needed: Voltage*Current=Power already resolves via vector addition in checkBinaryDimension"
  - "percent re-homed from the Mass hack onto a real Dimensionless dimension"
  - "Tesla unit named `Tesla` (not `T`) to avoid single-letter ambiguity; gauss/bar/Ci/rpm/g_accel shipped as non-SI convenience constructors with explicit factors"
  - "Action dimension (M·L²·T⁻¹) added here to support N-08 Planck constant"

requirements-completed: [REQ-phase-N-numeric-model (catalog content)]
adr-confirmation: "#1 (content half) — catalog declared, parses; sourced files carry NOTICE header"

# Metrics
completed: 2026-06-08
---

# Lane B / N-07: Physical-Quantity Catalog Expansion — Summary

**deal-stdlib quantity catalog grown from 15 to 58 dimensions and ~40 to 160 unit
constructors (ISO 80000 derived quantities), Dimensionless added with `percent` re-homed,
same-vector pairs declared distinctly and flagged for Lane A enforcement. Parse-verified on
the real toolchain.**

## What shipped

- **dimensions.deal** — +38 derived dimensions across space/time, mechanics, thermal,
  electromagnetism, photometry, physical chemistry, and ionizing radiation, each with a
  physics-verified `[M,L,T,I,TH,N,J]` vector. `Action` added for N-08.
- **dimensionless.deal** (new) — `Dimensionless`, `Angle`, `SolidAngle`, `Strain` (all-zero
  vectors, named-distinct) plus units `percent`, `rad`, `deg`, `sr`.
- **si_catalog.deal** (new) — 120 unit constructors: `Pa/kPa/MPa/GPa/bar`, `Hz…GHz`,
  `F/mF/uF/nF/pF`, `H/mH/uH/nH`, `Wb/Tesla/gauss`, `S/mS/uS`, areas/volumes,
  `m_per_s2/g_accel`, `rad_per_s/rpm`, densities, molar concentration, `kat`,
  `Gy/Sv/Bq/Ci`, `lx/lm`.
- **si.deal / mod.deal** — `percent` relocated; barrel re-exports the full new surface.

## Same-vector enforcement list (handed to Lane A / N-00 ③)

Declared distinct + `@same_vector_note`: Pressure/Stress; Torque/Energy;
Frequency/AngularVelocity/Radioactivity; HeatCapacity/Entropy; AbsorbedDose/DoseEquivalent;
Dimensionless/Angle/SolidAngle/Strain.

## Verification

- **`STDLIB_PARSE_OK`** — `cargo run -q -p deal -- parse` over every `units/*.deal` exits 0
  (real toolchain, confirmed by user 2026-06-08).
- Static lint (sandbox): braces balanced, 7-exponent/1-factor invariants hold, all literals
  valid, no keyword clashes, all barrel exports resolve.

## Deferred / hand-offs

- Cross-file `deal check` + `E2500` mismatch demonstration → blocked on **N-10** (CLI cross-file
  unit seeding). Probe model staged under `probes/` ready to run when N-10 lands.
- Torque≠Energy (and other same-vector) **enforcement** → Lane A sema, gated on **N-00 ③**.
- Full 270+ Modelica count → extend dimensions.deal/si_catalog.deal from `Modelica.Units.SI`
  with the same encoding (current core set is ISO 80000-complete for the showcase domains).
