# Source provenance

Ported from [BeZide93/ZeldaWWHDRecomp](https://github.com/BeZide93/ZeldaWWHDRecomp/tree/9efff3bfd76206da34adb0f96f2f556c410728e1/mods/hyrule-survivors), commit `9efff3bfd76206da34adb0f96f2f556c410728e1`, version 0.2.3.

The mod ID, options and data filenames are preserved. The catalogue compiles `src/mod.c` with the SDK pinned by the repository. Existing gameplay regression tests are retained under `tests/`. No runtime patch or binary from the recomp repository is included.

## Validation scope

The migrated entrypoint compiles against the unchanged SDK pin `fab80fcec42a25000a97137e2ee4eb2042b858c7`. USA and EU translation/native compilation and the retained synthetic regression checks were run locally. No real game files or device session were used for this migration. Existing gameplay limitations remain; package validation is not an in-game acceptance test.

The existing 0.2.3 Moblin crash mitigation and diagnostics are retained. This migration does not claim the reported Android crash is resolved.
