# Lane B acceptance probe — `precision-catalog.deal`

A ready-made consumer model that exercises the N-07 catalog and N-08 constants, and stages the
cross-file `E2500` mismatch check for the moment **N-10** (cross-file CLI wiring) lands.

## What it covers

- New N-07 derived quantities + constructors: `kPa`, `MPa`, `MHz`, `uF`, `L`, `m2`, `deg`, `percent`.
- The `Voltage * Current = Power` derived attribute (resolves via existing vector-addition algebra).
- N-08 explicit conversions across new domains: `to_Pa(psi(100))`, `to_m3(gal_us(50))`.
- `deal.std.constants` import (`c`, `k_B`, `g_0`).
- Three commented `// E2500-PROBE` mismatches, ready to uncomment after N-10.

## Running it

### 1. Syntactic parse — works TODAY (no resolution needed)

```bash
cd ~/projects/deal-lang/deal
cargo run -q -p deal -- parse \
  ../deal-stdlib/.planning/phases/0N-numeric-model/probes/precision-catalog.deal
# expect: exit 0
```

`deal parse` is purely syntactic, so this passes regardless of import resolution.

### 2. Semantic check — needs a workspace that resolves `deal-std`

`deal check` resolves imports, so the probe must run inside a project whose `deal.toml`
depends on `deal-std` (mirror `spec/examples/showcase/deal.toml`, which uses
`[dependencies] deal-std = "0.1"`). Drop `precision-catalog.deal` into that project's
`packages/` and:

```bash
deal check        # from the consumer project root
```

- **Today (N-07/N-08 done, N-10 not):** the `CatalogProbe` block is dimensionally valid.
  Cross-file dimensional verification is a graceful skip (imported units register as
  `.imported`, not `unit_def`), so no `E2500` fires even on a deliberate mismatch — this is
  the exact gap the N-10 todo closes.

### 3. After N-10 — confirm `E2500` fires

Once `analyzeWithExternalTable` is wired into the CLI `deal check` path (N-10), uncomment the
`MismatchProbe` block (or one `// E2500-PROBE` line at a time) and re-run `deal check`:

```
error[E2500] dimension mismatch
  --> precision-catalog.deal
   |  attribute bad1 : Pressure = MHz(10);
   |  Frequency value assigned to Pressure attribute
```

Each of the three probes should produce a non-zero exit with `error[E2500]`.

## Status

| Step | Gate | Status |
|------|------|--------|
| Syntactic parse | exit 0 | ✅ verified (part of `STDLIB_PARSE_OK`, 2026-06-08) |
| Semantic check, correct block | `deal check` exit 0 | ⏳ run in a deal-std consumer project |
| Cross-file `E2500` on mismatch | non-zero + `error[E2500]` | ⛔ blocked on N-10 |
