# deal-lang/deal-stdlib

Standard library for DEAL — Digital Engineering Authoring Language.

Version: v0.4.0 (Phase 4 — Ecosystem) · units catalog + constants expanded under Numeric-Model track Lane B (N-07/N-08)

## Shipped packages

```
deal-stdlib/
├── NOTICE.md                        # third-party attributions (Modelica, uom-rs, NIST)
├── packages/
│   ├── units/                       # SI + imperial units + ISO 80000 catalog
│   │   ├── dimensions.deal          # SI base + ~38 derived dimensions (7-exponent vectors)
│   │   ├── dimensionless.deal       # Dimensionless, Angle, SolidAngle, Strain, percent (N-07)
│   │   ├── si.deal                  # SI base + derived units with conversion factors
│   │   ├── si_catalog.deal          # derived-quantity unit constructors: Pa, Hz, F, H, … (N-07)
│   │   ├── imperial.deal            # Imperial units (SP 811)
│   │   ├── conversions.deal         # explicit to_<SI>() conversion call forms (N-08)
│   │   └── index.deal                 # Barrel export
│   ├── constants/                   # CODATA 2022 fundamental constants (N-08)
│   │   ├── codata.deal              # c, h, k_B, N_A, q_e, G, eps_0, … 
│   │   └── index.deal
│   └── interfaces/                  # Electrical + mechanical interfaces — shipped Phase 4
│       ├── electrical/
│       │   ├── rj45.deal            # Ethernet RJ45 interface
│       │   ├── usb_c.deal           # USB-C connector
│       │   ├── can.deal             # CAN bus interface
│       │   ├── rs422.deal           # RS-422 serial
│       │   └── index.deal
│       └── mechanical/
│           ├── bolt_patterns.deal
│           └── index.deal
├── deal.toml
└── README.md
```

## Deferred packages (Phase 6 — stdlib expansion)

The following packages are planned for Phase 6. They are NOT included in Phase 4.
Do not import from these packages in Phase 4 DEAL models.

```
packages/
├── rf/                    # Phase 6 — deferred
│   ├── sma.deal           # SMA RF connector
│   ├── n_type.deal        # N-type RF connector
│   └── index.deal
├── protocols/             # Phase 6 — deferred
│   ├── mil_std_1553.deal  # MIL-STD-1553 data bus
│   ├── arinc_429.deal     # ARINC 429 avionics bus
│   ├── spacewire.deal     # SpaceWire
│   ├── http.deal          # HTTP protocol
│   ├── mqtt.deal          # MQTT IoT protocol
│   └── index.deal
├── standards/             # Phase 6 — deferred
│   ├── mil_std_810h.deal  # Environmental test methods
│   ├── do_178c.deal       # Software assurance levels
│   ├── do_254.deal        # Hardware assurance levels
│   └── index.deal
└── patterns/              # Phase 6 — deferred
    ├── redundancy.deal    # TMR, dual-redundancy patterns
    ├── watchdog.deal      # Watchdog timer patterns
    └── index.deal
```

## Usage

```deal
import deal.std.units.{kg, m, s, V, A, kWh, degC, ohm};
import deal.std.units.{Mass, Length, Time, Current, Temperature, Energy, Voltage};
import deal.std.interfaces.electrical.{RJ45, USBC};
```

## Dimensional metadata encoding

Unit and dimension definitions use the `attribute def` body encoding (ADR: `ADR-phase-4-dimension-metadata-syntax.md`).

Each dimension carries a 7-exponent SI base vector:
```deal
attribute def Mass {
    attribute si_M  : Integer = 1;   // mass exponent
    attribute si_L  : Integer = 0;   // length exponent
    attribute si_T  : Integer = 0;   // time exponent
    attribute si_I  : Integer = 0;   // current exponent
    attribute si_TH : Integer = 0;   // temperature exponent
    attribute si_N  : Integer = 0;   // amount exponent
    attribute si_J  : Integer = 0;   // luminosity exponent
}
```

Each unit carries `si_factor` (scale relative to SI base unit):
```deal
attribute def lb <<specializes>> Mass {
    attribute si_factor : Real = 0.453592;
}
```

## Explicit conversions (D-57)

Mixed-unit same-dimension expressions require explicit conversion calls:
```deal
// ERROR — lb and kg are both Mass but different units
// attribute gross : Mass = lb(3300) + kg(1500);   // E2501

// CORRECT — explicit conversion to common unit
attribute gross : Mass = to_kg(lb(3300)) + kg(1500);
```

Conversion functions are imported from `deal.std.units`: `to_kg`, `to_m`, `to_s`, `to_N`, `to_J`, `to_W`, `to_K`, `to_m_per_s`, plus the N-08 additions `to_Pa`, `to_m2`, `to_m3`, `to_Hz`.

## SI canonical, imperial opt-in (SL-7)

SI units are canonical and (per SL-3) available without restriction. Imperial units and
the `to_<SI>()` converters require an explicit import — a model that uses only SI never
pulls in imperial definitions:

```deal
import deal.std.units.{lb, ft, to_kg, to_m};   // imperial: explicit opt-in
attribute gross : Mass = to_kg(lb(3300)) + kg(1500);
```

## Physical constants (CODATA 2022)

Constants live in `deal.std.constants` and use the unit-style call form — `c(1)` yields the
constant value, `c(2)` is 2× it (same convention as `kg(1500)`):

```deal
import deal.std.constants.{c, k_B, N_A};
attribute lightSpeed : Speed = c(1);           // 2.99792458e8 m/s
```

Defined constants (c, h, k_B, N_A, q_e, g_0) are exact per the 2019 SI redefinition; measured
constants (m_e, m_p, G, eps_0, mu_0) carry a `// precision:` note with the CODATA 2022 relative
standard uncertainty. Re-verify last digits against physics.nist.gov before any release tag.

## Dependencies

- None (pure DEAL definitions)

## Version scheme

deal-stdlib versioning locksteps with the deal toolchain phase:
- `v0.4.0` — Phase 4 (Ecosystem): units + interfaces packages
- `v0.4.x` — Numeric-Model track Lane B: ISO 80000 catalog expansion (N-07), CODATA 2022
  constants + SP 811 completion (N-08). Attribution in `NOTICE.md`.

## References

- BIPM SI Brochure 9th edition (2019) — SI base and derived units
- ISO 80000-1 / ISO 31 — quantities and units (derived-quantity catalog)
- NIST SP 811 (2008) — imperial unit conversion factors
- NIST CODATA 2022 — fundamental physical constants (https://physics.nist.gov/cuu/Constants/)
- Modelica Standard Library (BSD-3) / uom-rs (Apache-2.0) — see NOTICE.md
