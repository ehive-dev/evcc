# FNN snapshot API v1

`GET /api/fnn` is a loopback-only endpoint for external-limit consumers. It
returns one jointly acquired DI1/DI2 sample, rather than independently refreshed
contacts. Example response:

```json
{"version":1,"sequence":42,"valid":true,"sampled_at":"2026-11-22T12:00:00.000Z","age_ms":250,"di1":true,"di2":false}
```

Both lines are read in one `gpioget` invocation. Optional multi-sample filtering
selects an observed pair, preserving that pair's acquisition time. `sequence`
increments on successful acquisition only. `sampled_at` is its wall-clock time;
`age_ms` uses a monotonic clock and increases even if wall time changes. Serving
an HTTP response never refreshes acquisition time. Overlapping polls are skipped.

Before the first successful sample, after a GPIO read/parse failure, or when the
sample is older than `max(5000, 3 * POLL_MS)` milliseconds, the endpoint returns
HTTP 503 with `valid: false`. A later successful acquisition restores validity.
All responses use `Cache-Control: no-store`. Retained contact values in invalid
responses are diagnostic only and must not be applied by consumers.

Consumers must reject HTTP errors, unsupported versions, `valid != true`, missing
or non-boolean selected inputs, invalid sequence numbers and missing, negative
or excessive `age_ms`. Disable HTTP caching. On a failed update, keep the last
accepted limit and report an error; fail startup if there is no valid initial
sample. This API does not define a substitute limit for an extended outage.

The evcc `ehive-one-fnn` template requires this snapshot API (the previously
released ioManager 1.1.2 does not provide it). Its default maximum age is 5000 ms;
set `maxagems` above the configured acquisition interval. The default acquisition
interval is now 1000 ms, matching the packaged configuration. Existing
`/api/gpio` and `/api/status` responses remain available; their timestamps and
legacy cached booleans are not freshness evidence for FNN control.
