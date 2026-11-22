# eHive FNN review verification, 2026-11-22

The three findings from the first review are implemented and regression-tested.
This report covers the revised source and producer contract, superseding the
freshness limitation of the initial report for this test build only.

## Source and build provenance

- Public evcc source: `09aa0deb2e6d7c73d282b6ae87309481ea4f9d3f`.
- evcc test binary: `0.317.1-ehive.3`, SHA-256 `e01853290070ef82c7b0e7fcb8343e3acfe78dfab7e54374280eec4a0a73716b`.
- Built from local commit `54dfbc70a2ed658adb9a201632defc55e19d4755`.
  Its source tree `f8a0045f98ba2e057442e56706457085acf4027e` is identical
  to the public commit's tree. The test binary reuses the existing 0.317.1 UI
  assets; this work changes backend/template behavior only.
- ioManager producer source: `38b70a9a6192bfc3492b35d6551e79d3dad5029d`.
  Bundle SHA-256 `4491db066058c34ffe5952f5bd49e2355a6e18d4f4c21705c2a72cc239c3f8dd`.
  This producer is a development build, not a released ioManager version.

## Review regressions

- HTTP/FNN tests reject missing, invalid, non-boolean and stale selected input
  data; HTTP 503 and validation errors retain the last accepted limits.
- One HTTP response supplies every mapped contact in an evaluation. Alternating
  source responses verify one fetch at startup and one per control cycle.
- A legacy W3-active/W4-error startup creates no grid sessions. Runtime W4
  failures leave both previously accepted limits and sessions unchanged.
- Schema changes, unknown contacts, mixed snapshot/individual configuration,
  invalid metadata, priority, polarity and release are covered.
- HEMS and template package tests, template documentation generation and the
  FNN package with Go's race detector completed successfully.
- All 31 ioManager tests completed, including acquisition age, joint reads,
  observed-pair filtering, overlapping acquisition, HTTP errors and recovery.

## Physical contact test

On the ARM64 eHive development device, two Shelly Plus 1 dry contacts drove the
actual DI1 and DI2 lines. GPIO readback, snapshot data and running FNN status
were compared. Separate evcc instances used a constant virtual grid and no
loadpoints or inverters.

- 320 mapping/polarity/contact combinations passed.
- 18 running-controller states passed for W4/W3 and S1/S2.
- Five duplicate/empty mappings were rejected.
- DI2 selection in the legacy API now works in the candidate producer.
- Relays returned to their original OFF/OFF state.
- The real-consumer evcc process and its configuration remained unchanged.
- The development device's ioManager was switched to the candidate using a
  separate service override and backup. Its packaged original bundle remains.

A separate producer process first read the real GPIO, then received injected
command failures. It returned HTTP 503, retained the old acquisition timestamp
and sequence, increased age, and recovered only after a successful GPIO read.
This did not restart either main service. The full summary includes these
snapshots. Automated clock tests also verify age expiry independently of wall
clock time and HTTP response time.

## Remaining scope

The existing grid-session history discrepancy is still reproduced: an active
60% to 30% transition updates live HEMS limits correctly but retains the initial
limit in the session history record. This is recorded as a remaining finding;
`template_passed` is true while `all_findings_closed` is false. It is separate
from the three review findings and is not hidden by the test result.

No actual wallbox/inverter power response or regulatory conformity was tested.
The new template requires the snapshot API described in `fnn-snapshot-api.md`;
released ioManager 1.1.2 lacks that API. A producer release remains necessary
before normal installation. Upstream CI and maintainer acceptance are pending.
