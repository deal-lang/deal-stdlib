---
track: 0N-numeric-model
lane: B
plan: "08"
subsystem: stdlib
tags: [deal-stdlib, codata-2022, constants, sp811, conversions, sl-7, parse-verified]

# Dependency graph
requires:
  - plan: 0N-09
    provides: NOTICE.md + header template (CODATA/SP 811 public-domain attribution)
  - plan: 0N-07
    provides: derived dimensions (Pressure, Area, Volume, Frequency, Permittivity, ...) the
              constants and converters specialize
provides:
  - deal-stdlib deal.std.constants package (codata.deal, mod.deal) — 14 CODATA 2022 constants
  - deal-stdlib conversions.deal — to_<SI>() forms relocated from imperial.deal + 4 new domains
  - deal-stdlib imperial.deal completed to NIST SP 811 (psi, atm, gal_us, nmi, acre, degR, ...)
  - SL-7 documented (SI canonical, imperial opt-in) in README + conversions.deal
affects: [0N-11-integration, Lane-A-N-01 (precision-note upgrade)]

# Tech tracking
patterns-established:
  - "Constant encoding: unit-style def with si_factor = SI value; call form c(1) yields the constant"
  - "Niche constant-support dimensions (PerAmount, GravConstantDim, MolarGasDim, StefanBoltzmannDim) kept local to codata.deal"
  - "Measured constants carry `// precision:` note with CODATA relative uncertainty (upgrade to ± e-N after Lane A N-01)"
  - "Scientific-notation Real literals (6.62607015e-34) — valid; lexer requires a decimal point before the exponent"

key-files:
  created:
    - packages/constants/codata.deal
    - packages/constants/mod.deal
    - packages/units/conversions.deal
  modified:
    - packages/units/imperial.deal   # +12 SP 811 units; converters removed (relocated)
    - packages/units/mod.deal        # imperial additions; converters now from conversions.deal
    - README.md                      # SL-7, constants usage, package listing, references
    - deal.toml                      # description updated; packages/* glob already covers constants/

key-decisions:
  - "Constants as unit-style defs (c(1) call form) — only parse-safe form under the locked grammar; bare-value constant syntax deferred to Lane A N-01"
  - "CODATA 2022 values web-verified for measured constants (m_e, m_p, eps_0, mu_0, G) — 2022 not 2018 figures"
  - "Exact constants (c, h, k_B, N_A, q_e, g_0, R, sigma) flagged exact per 2019 SI redefinition"
  - "Elementary charge named q_e (not e) to avoid confusion with exponent notation in expressions"
  - "Converters relocated to conversions.deal; mod.deal sources them there; imperial.deal keeps a pointer note"
  - "SL-7: imperial + converters require explicit import; SI never pulls in imperial"

requirements-completed: [REQ-phase-N-numeric-model (constants + conversions content)]

# Metrics
completed: 2026-06-08
sources:
  - "NIST CODATA 2022 wallet card — https://physics.nist.gov/cuu/pdf/wallet_2022.pdf"
  - "CODATA 2022 (arXiv:2409.03787)"
  - "NIST SP 811 (2008) — conversion factors"
---

# Lane B / N-08: Constants & Conversions — Summary

**deal.std.constants ships 14 CODATA 2022 fundamental constants; imperial completed to NIST
SP 811; to_<SI>() converters relocated and extended; SL-7 documented. Parse-verified on the
real toolchain.**

## What shipped

- **constants/codata.deal** (new package `deal.std.constants`) — c, h, hbar, k_B, N_A, q_e,
  g_0, R, sigma (exact) and m_e, m_p, G, eps_0, mu_0 (measured, with `// precision:` notes).
  Four niche support dimensions defined locally.
- **conversions.deal** (new) — 8 relocated converters + `to_Pa`, `to_m2`, `to_m3`, `to_Hz`.
- **imperial.deal** — +`nmi`, `psi`, `ksi`, `inHg`, `atm`, `acre`, `gal_us`, `qt`, `pt`,
  `floz`, `ftlb`, `degR`; converters removed (relocated).
- **README / deal.toml / mod.deal** — SL-7 documented, constants usage shown, version note added.

## Verification

- **`STDLIB_PARSE_OK`** — `cargo run -q -p deal -- parse` over every `units/*.deal` and
  `constants/*.deal` exits 0 (real toolchain, confirmed by user 2026-06-08).
- Static lint (sandbox): 212 defs total — braces, scientific-notation literals (against the
  lexer's decimal-point rule), specializations, imports, and barrel exports all resolve.

## Deferred / hand-offs

- `// precision:` notes on measured constants → upgrade to `± e-N` annotations once Lane A
  **N-01** ships the precision grammar.
- Cross-file `deal check` resolution of `deal.std.constants` imports → exercised by the probe
  once **N-10** wires cross-file seeding.
- Bare-value constant form (`= c` instead of `= c(1)`) → optional Lane A enhancement.
- Re-verify constant last digits against physics.nist.gov before any release tag.
