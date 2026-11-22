# ioManager snapshot API release, 2026-11-22

The required snapshot producer is now available in stable
[ioManager 1.1.3](https://github.com/ehive-dev/iomanager-releases/releases/tag/v1.1.3).
The release includes the ARM64 Debian package, build provenance and SHA256SUMS.
It carries its own pinned Node.js runtime and requires no specific SmartHub version.

The package was built offline in a network namespace with loopback only from
source commit `e69929352b4a7446c215e07b537b5778491b9dee`. All 31 native ARM64 tests
completed. The actual packaged runtime and bundle were exercised read-only,
then installed and tested on the ARM64 eHive development device. Five physical
contact states were checked before installation and five afterward, including
legacy DI1 value behavior, DI2 selection, invalid selectors and snapshot API v1.
The normal package post-installation procedure also completed successfully.

Existing settings were preserved. The package-owned runtime and installed bundle
were checked against the package. The manual producer override was removed;
the development device now runs the regular installed 1.1.3 package. Both relays
were restored OFF and the main evcc process and configuration remained unchanged.

The [held-contact cache report](held-contact-report.md) still applies to the
unchanged consumer candidate. It documents normal held-contact operation and
the artificial delayed-response case. The time-based cache remains non-atomic;
release of the producer does not remove that documented limitation.
The generic FNN startup-validation fix is outside the current template-only PR.

The full build provenance is available as `producer-1.1.3-build-info.json` and as
a release asset. The package SHA-256 is
`509cfc6d08eb6f4d4fa860125132b8a3cecc7ced4447c16077d2572a4b920030`.
