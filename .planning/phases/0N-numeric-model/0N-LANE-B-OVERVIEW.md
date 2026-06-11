---
track: 0N-numeric-model
lane: B (stdlib content)
type: overview
status: ready
depends_on: []
source_adr: deal/.planning/decisions/ADR-deal-stdlib-numeric-model.md
roadmap: DEAL-STDLIB-NUMERIC-ROADMAP.html
plans:
  - 0N-09-PLAN.md   # NOTICE & attribution    (do first)
  - 0N-07-PLAN.md   # catalog expansion       (start in parallel)
  - 0N-08-PLAN.md   # constants & conversions (start in parallel)
---

# Lane B — Physical-Quantity Stdlib Content

Lane B delivers the **content** half of the ADR numeric model: the ISO 31 / ISO 80000
quantity catalog, physical constants, the full conversion table, and the licensing
attribution. None of it requires a compiler change, so Lane B runs **fully in parallel**
with the Lane A compiler spine and with mainline Phase 5/6 work. All authoring uses the
already-locked Option A encoding (ADR-phase-4-dimension-metadata-syntax) and is verified
by the existing `deal parse` / `deal check` — no new grammar.

## Dependency decision (N-07 → N-00 ③)

**Resolved: split N-07; the Torque≠Energy enforcement is NOT Lane B work.**

- The current Check #7 algebra (`sema.zig`) compares only the 7-exponent vector. Torque and
  Energy are both `M·L²·T⁻²`, so the compiler already treats them as identical. **Declaring**
  Torque, Energy, Angle, SolidAngle, etc. as distinct *named* dimensions is pure content and
  parses + checks today.
- **Enforcing** that a Torque can't be assigned to an Energy attribute requires a named-type
  tie-break in `sema.zig`. That is a Lane A sema feature, gated on N-00 ③ — it is reclassified
  out of Lane B.
- Common derived products (`Voltage*Current=Power`) already resolve via vector addition in
  `checkBinaryDimension`; **no dimension-equation table is needed** for them. A declared table
  is only relevant to same-vector disambiguation, i.e. the Lane A enforcement above.

**Net effect:** Lane B has **zero blocking dependencies** and starts immediately. N-07 declares
every named quantity (including the same-vector pairs) with a `same_vector_note` doc comment so
the later Lane A enforcement pass has an unambiguous list of which names must stay distinct.

## Sequencing

```
N-09 (NOTICE + header template)   ── do FIRST (½ day) ──┐
                                                        ├──► N-07 and N-08 author files
N-07 (catalog)   ─┐                                     │     WITH headers already required
N-08 (constants) ─┴─ run in parallel after N-09 lands  ◄┘
```

N-09 is first only so that N-07/N-08 land their sourced files with the required copyright
header already in place (avoids a retro-fit pass). N-07 and N-08 are mutually independent.

## Target file layout (after Lane B)

```
deal-stdlib/
├── NOTICE.md                                  # N-09 (new)
├── packages/
│   ├── units/
│   │   ├── dimensions.deal                    # N-07: expand 15 → ~80 dimensions
│   │   ├── dimensionless.deal                 # N-07 (new): Dimensionless, Angle, SolidAngle
│   │   ├── si.deal                            # N-07: keep; percent moves out
│   │   ├── si_catalog.deal                    # N-07 (new): derived-quantity unit constructors
│   │   ├── imperial.deal                      # N-08: complete SP 811 table
│   │   ├── conversions.deal                   # N-08 (new): converter forms, all domains
│   │   └── mod.deal                           # N-07/08: extend barrel export
│   └── constants/                             # N-08 (new package)
│       ├── codata.deal                        # CODATA 2022 fundamental constants
│       └── mod.deal
└── deal.toml                                  # N-08: register constants package in [workspace]
```

All quantity files keep `package deal.std.units;`. Constants use `package deal.std.constants;`.

## Encoding rules (locked — ADR-phase-4-dimension-metadata-syntax, Option A)

Dimension def — declares all seven `si_M..si_J` Integer exponents, no `<<specializes>>`:

```deal
/** Pressure — M·L⁻¹·T⁻². SI derived unit: pascal (Pa = N/m²). */
attribute def Pressure {
    attribute si_M  : Integer = 1;
    attribute si_L  : Integer = -1;
    attribute si_T  : Integer = -2;
    attribute si_I  : Integer = 0;
    attribute si_TH : Integer = 0;
    attribute si_N  : Integer = 0;
    attribute si_J  : Integer = 0;
}
```

Unit def — `<<specializes>>` a dimension, declares `si_factor : Real`:

```deal
/** Pa — pascal, SI derived unit for Pressure. Factor: 1.0. */
attribute def Pa <<specializes>> Pressure { attribute si_factor : Real = 1.0; }
/** kPa — kilopascal. Factor: 1000 Pa. */
attribute def kPa <<specializes>> Pressure { attribute si_factor : Real = 1000.0; }
```

Same-vector named distinction (flag for Lane A enforcement):

```deal
/** Torque — M·L²·T⁻². Shares the Energy vector; kept distinct by name.
 *  @same_vector_note: distinct-from Energy — Lane A enforcement (N-00 ③). */
attribute def Torque {
    attribute si_M  : Integer = 1;  attribute si_L  : Integer = 2;
    attribute si_T  : Integer = -2; attribute si_I  : Integer = 0;
    attribute si_TH : Integer = 0;  attribute si_N  : Integer = 0;
    attribute si_J  : Integer = 0;
}
```

## Verification (oracle = the parser; same as Plan 04-03)

```bash
cd /Users/dunnock/projects/deal-lang/deal
for f in ../deal-stdlib/packages/units/*.deal ../deal-stdlib/packages/constants/*.deal; do
  cargo run -q -p deal -- parse "$f" >/dev/null || { echo "PARSE FAIL: $f"; exit 1; }
done && echo STDLIB_PARSE_OK
```

A `deal check` pass over a small consumer model that imports the new units and writes a
dimensionally-correct expression must exit 0; a deliberate mismatch must emit `E2500`.

## Lane B exit gate (rolls into roadmap N-11)

1. `STDLIB_PARSE_OK` — every `.deal` in `units/` and `constants/` parses, exit 0.
2. ADR Confirmation #1 (content half): the expanded catalog covers the ISO 80000 quantities
   enumerated in 0N-07 §Catalog, each sourced def carries a BSD-3 header.
3. ADR Confirmation #6: `NOTICE.md` present at repo root with the Modelica BSD-3 + uom-rs
   Apache-2.0 + NIST public-domain notices verbatim from the ADR.
4. `percent` resolves as `Dimensionless`, not `Mass`.
5. A consumer model importing `Pressure`/`Frequency`/`Capacitance` units `deal check`s clean;
   a mismatch emits `E2500`.

See `0N-09-PLAN.md`, `0N-07-PLAN.md`, `0N-08-PLAN.md` for task-level detail.
