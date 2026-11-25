# Local workflow

Repository: `clodhub/windows-desktop`. Licence: `Apache-2.0` (`LICENCE`, `NOTICE`).

Required-local suite from the package root: `node --test tests/policy.test.cjs tests/result.test.cjs tests/runtime.test.cjs tests/package-staging.test.cjs` (also `npm test`).
Platform suites (heavy; install Electron over the network; do not run casually): `node scripts/platform-check.cjs package`, `node scripts/platform-check.cjs runtime` (Windows x64 only).
Review a fetched archive: `python3 -m aepm review <digest> --store STORE --policy POLICY --trusted --allow-network`. Always `-B`.
