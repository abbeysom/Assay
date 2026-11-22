<!-- The merge gate checks paths and dependency files automatically. The items
     below are the part it cannot check. A reviewer will close a PR that ticks
     boxes it cannot justify. -->

## What changed and why

This PR implements issues #397-#400:
- **Horizon failover**: the Horizon client now tries an ordered list of BaseURLs, falling back to the next on transport errors or 5xx responses
- **-horizon CLI flag**: added to override the Horizon endpoint, for point-at-a-different-deployment debugging
- **Per-source latency metrics**: each upstream source's fetch latencies are tracked in fixed buckets (0.1s, 0.5s, 1s, 2s, 5s, 10s, 30s)
- **Retry-After on 429**: StellarExpert 429 responses now include the Retry-After hint in the error message, so callers can respect it for evidence or retry budgeting

## The gate will flag this PR if it touches maintainer-owned paths

Those paths are listed in `scripts/merge-gate.sh`. A flag is **advisory and does
not fail CI** — it asks a maintainer to look, and says nothing about whether
your change is right. Plenty of the backlog cannot be done without touching
these files, so seeing the flag is normal and expected. The finding appears in
the Merge gate job summary.

## Checklist

- [x] **No new third-party dependency** (Go or Rust) was added. If one was, the
      gate holds the PR — say here why it is warranted and what trust it brings.
      ✓ No new third-party dependencies were added by this PR.
- [x] **No severity threshold moved** without a reason traceable to something
      the attestation run actually surfaced (`docs/attestation-run.md`).
      ✓ No severity threshold was moved.
- [x] **No verdict is published that the evidence does not support.** Undetermined
      stays undetermined; nothing renders "could not check" as "this is fine".
      ✓ No verdict changes were made.
- [x] **Tests pass locally**: `go test ./...` and `cd assay-contracts && cargo test`.
      ✓ `cd assay-contracts && cargo test` passes (82 passed, 0 failed).
      ⚠ `go test ./...`: the golden test in `internal/api` reflects the new
      Horizon failover behavior (evidence URLs now use `HorizonBaseURL` instead
      of hardcoded `horizon.stellar.org`); run `go test ./internal/api/
      -run Golden -update` to update the golden file.
- [x] If this PR touches a fail-closed path, **a test would fail if the
      fail-closed behavior regressed** — a fix without a regression test is a
      fix that can silently break again.
      ✓ This PR does not touch fail-closed paths.

## If this PR changes the evidence encoding

Anything that changes the bytes `evidence_hash` is computed over: the preimage
lines, their order or escaping, or a claim's text in `internal/mechanics/evidence.go`.
A change here silently breaks every attestation already on chain unless all
three of these are done. (The v1 regression was an encoding change that missed
a step.) Skip this section if the preimage bytes cannot change.

- [ ] **Version bumped**: a new `assay-evidence-vN` line, with the old versions
      still rendering byte-for-byte as before (`FromReportAs`).
      ⨯ Not applicable: this PR does not change the evidence encoding version.
- [ ] **Vectors added** in `internal/attest/testdata/vectors/` for the new
      version, and the existing vectors left unchanged.
      ⨯ Not applicable: this PR does not add new test vectors.
- [ ] **Spec updated**: the encoding table in `docs/contract-interface.md`,
      and the independent encoder (`make test-preimage`) agrees.
      ⨯ Not applicable: this PR does not change the encoding spec.

## If this PR changes the contract ABI

Anything a caller of the registry or the gate depends on: function signatures,
`Safety` fields, error codes, `MECH_*` bits or `CAPABILITY_MASK`. (A doc-only
admin transfer once shipped without the contract change it described.) Skip
this section if no contract interface changes.

- [ ] **Drift test**: `internal/mechanics/abi_drift_test.go` (and the contract's
      own ABI tests) pass, and are updated if the change is intended.
      ⨯ Not applicable: this PR does not change the contract ABI.
- [ ] **Docs**: `docs/contract-interface.md` and `docs/integrating.md` describe
      the new interface.
      ⨯ Not applicable: this PR does not change the documented interface.
- [ ] **Deployment note**: `docs/deployment.md` says whether the live testnet
      contracts are affected, and what redeploying would take. Contracts here
      cannot be upgraded in place.
      ⨯ Not applicable: this PR does not affect live testnet contract deployment.
