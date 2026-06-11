---
track: 0N-numeric-model
lane: B
plan: 09
type: execute
wave: 0
depends_on: []
autonomous: true
files_modified:
  - NOTICE.md
  - packages/units/HEADER-TEMPLATE.txt
requirements:
  - REQ-phase-N-numeric-model
source_adr: deal/.planning/decisions/ADR-deal-stdlib-numeric-model.md
adr_confirmation: "#6"
must_haves:
  truths:
    - "NOTICE.md exists at the deal-stdlib repository root carrying the Modelica BSD-3, uom-rs Apache-2.0/MIT, and NIST public-domain notices verbatim from the source ADR"
    - "A per-file copyright header template exists for sourced catalog/constants files (N-07, N-08)"
  artifacts:
    - path: "NOTICE.md"
      provides: "Licensing attribution required by ADR Confirmation #6"
      contains: "Modelica Association"
    - path: "packages/units/HEADER-TEMPLATE.txt"
      provides: "Copy-paste copyright header block for files derived from Modelica / uom-rs"
---

<objective>
Discharge the ADR's attribution obligation before any sourced content lands. Create the
root `NOTICE.md` with the three required attributions verbatim, and a short header template
that N-07 and N-08 paste into each file derived from an external source. This is the smallest,
zero-dependency plan in Lane B and is sequenced first so downstream files are born compliant.

Output: `deal-stdlib/NOTICE.md`, `deal-stdlib/packages/units/HEADER-TEMPLATE.txt`.
</objective>

<context>
The ADR "Sourcing Attribution Requirements" section mandates exact text. Do not paraphrase —
copy it. Confirmation #6 checks the Modelica + uom-rs notices are present.
</context>

<task id="09-1" name="Author NOTICE.md">
Create `deal-stdlib/NOTICE.md` with this content verbatim (from the ADR):

```
# NOTICE — deal-stdLib third-party attributions

deal-stdLib type name catalog derived in part from the Modelica Standard
Library, Copyright (c) 1998-2020, Modelica Association and contributors.
Used under the BSD 3-Clause License.
https://github.com/modelica/ModelicaStandardLibrary/blob/master/LICENSE

deal-stdLib dimension vector concept derived in part from uom-rs,
Copyright (c) iliekturtles contributors.
Used under the Apache License 2.0 / MIT License.
https://github.com/iliekturtles/uom

Physical constants sourced from NIST CODATA 2022.
Unit conversion factors sourced from NIST SP 811.
Both are in the public domain.
```

Add nothing beyond a one-line title; the body must match the ADR text.
</task>

<task id="09-2" name="Author header template">
Create `deal-stdlib/packages/units/HEADER-TEMPLATE.txt` — the comment block to place at the
top of any file containing Modelica-derived type names (N-07 `dimensions.deal`,
`si_catalog.deal`) or CODATA/SP 811 values (N-08):

```
// ─────────────────────────────────────────────────────────────────────────
//  Portions of this file derive from the Modelica Standard Library
//  (BSD 3-Clause, (c) 1998-2020 Modelica Association) and the ISQ dimension
//  vector concept from uom-rs (Apache-2.0 / MIT). See deal-stdlib/NOTICE.md.
//  Quantity values: NIST CODATA 2022 / NIST SP 811 (public domain).
// ─────────────────────────────────────────────────────────────────────────
```

Place it immediately after the existing `@header { ... }` block, before `package …;`.
</task>

<verification>
<automated>test -f /Users/dunnock/projects/deal-lang/deal-stdlib/NOTICE.md && grep -q "Modelica Association" /Users/dunnock/projects/deal-lang/deal-stdlib/NOTICE.md && grep -q "iliekturtles" /Users/dunnock/projects/deal-lang/deal-stdlib/NOTICE.md && grep -q "CODATA 2022" /Users/dunnock/projects/deal-lang/deal-stdlib/NOTICE.md && echo NOTICE_OK</automated>
- `NOTICE.md` present at repo root; contains all three attributions (Modelica, uom-rs, NIST).
- `HEADER-TEMPLATE.txt` present for N-07/N-08 to consume.
</verification>

<acceptance>
ADR Confirmation #6 satisfied. Downstream plans paste the header into every sourced file.
</acceptance>
