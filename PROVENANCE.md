# Source provenance

The initial relay source is an exact subtree extraction from the Apache-2.0
licensed application repository `Xare123/openbubbles-app`, commit
`161f35952c22cb59b8cb3a034e176097f2df8b60`, directory
`tooling/ios/native-validation-relay`.

`SOURCE_IMPORT.json` records each original Git blob and SHA-256. Runtime source,
entitlements, build scripts, and tests are unchanged by this extraction. The two
historical README/provenance files were updated to describe the now-observed
results and standalone paths. New publication documentation, ignore rules, and
a manual build workflow are the only additions. The root `LICENSE` is copied
verbatim from that application commit.

The account generators were independently implemented and adapted from the
project's earlier on-device diagnostic work. Device validation uses an
independently authored native adapter and interface declaration. No legacy
ValidationRelay Swift implementation or SDK-generated Mach client is copied.
The functional interface references and SDK generation requirements are retained
in [nac/PROVENANCE.md](nac/PROVENANCE.md).

The original `JJTech0130/ValidationRelay` repository did not declare a repository
license when checked for this publication. This extraction does not grant a
license to that repository, republish its source, or replace the existing
`Xare123/ValidationRelay` fork. GitHub will report this new extraction as a
standalone repository rather than as part of that older fork network.

Only source and synthetic tests belong here. Device evidence, runtime
registrations, account inputs, screenshots, phone numbers, local databases,
diagnostic applications, USB-control helpers, and signed client APKs are excluded.
