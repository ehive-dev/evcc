# eHive One FNN hardware validation

Hardware test date: 21 November 2026. Hardware build: evcc 0.317.1-ehive.2.
The contribution has additionally been checked against current upstream master;
the hardware run is not presented as a hardware test of that newer master.

Two Shelly Plus 1 dry-contact fixtures were connected to the two digital inputs
of an eHive One. Each relay state was checked independently against direct GPIO
reads and the local ioManager API. The template uses the explicit boolean fields
`di1` and `di2` from that API.

| Check | Result |
| --- | --- |
| 20 valid mappings × 4 polarity combinations × 4 physically switched input states | 320/320 |
| Running FNN controller: W4/W3 activation, simultaneous limits and release | 9/9 states |
| Running FNN controller: S1/S2 priority, 60↔30% transitions and release | 9/9 states |
| Empty assignment and four duplicate signal assignments | 5/5 rejected |
| Fixture after test | Both relays OFF; both GPIOs 0; both API fields false |
| Existing main evcc process/configuration | Unchanged |

The mapping tests used the normal authenticated configuration-test API of a
temporary evcc instance. The dynamic tests used running FNN instances and read
the current `hems.status`. Each instance had its own database and a constant
virtual grid meter, without configured consumers or inverters. Temporary services
were stopped after the test.

Test values were 4200 W for consumption dimming and 10000 Wp for generator power.
The largest observed update times in these runs were 1.189 s at the ioManager
and 5.045 s at the live HEMS state. These are measurements, not guaranteed timings.
Actual wallbox/inverter power changes were not tested.

## Open findings and limitations

- The installed ioManager 1.1.2 did not apply `input=di2` to its legacy `value`
  field. Its explicit `di1`/`di2` fields were correct; this template uses those
  fields and is unaffected by the selector issue.
- During an active 60↔30% curtailment transition, the live HEMS limit updated
  correctly but the grid-session record retained its initial power. Three
  transition states showed this reporting discrepancy. It is separate from
  the input mapping/template change.
- The ioManager response has no sample-age/validity metadata. Strict boolean
  parsing detects malformed responses, but cannot detect stale valid booleans.
  The release acceptance contract for stale data remains open.

Accordingly, the machine-readable report distinguishes `template_passed: true`
from `all_checks_passed: false`. This report does not establish complete system
acceptance or EEBUS interoperability.

The [sanitized machine-readable summary](hardware-summary.json) contains the
320 mapping results and 18 dynamic observations. Local addresses, host paths,
serial numbers, credentials and databases are excluded.

🤖 Prepared with [OpenAI Codex](https://openai.com/codex/)
