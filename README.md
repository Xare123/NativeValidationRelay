# Native Validation Relay

An experimental, native iPhone relay for a matching OpenBubbles client. It keeps
device validation, native Apple Account authentication, and native push signing
as separate operations instead of substituting one proof for another.

This repository extracts the working relay from
[Xare123/openbubbles-app at `161f35952`](https://github.com/Xare123/openbubbles-app/tree/161f35952c22cb59b8cb3a034e176097f2df8b60/tooling/ios/native-validation-relay).
It is an independently implemented replacement, **not a GitHub-network fork of
the older ValidationRelay implementation**. The older repository remains
unchanged. See [provenance](PROVENANCE.md) and [license](LICENSE).

## What the repair includes

- Genuine on-device NAC device validation, separately from the account qualifier.
- Native account headers and an empty-payload account qualifier generated on the
  iPhone, with an explicit capability signal to the compatible client.
- Native APS identity discovery and bounded push/registration signing. The
  private APS key stays on the iPhone.
- Validated registration persistence, connection-bound replies, reconnect handling,
  and app-local keep-awake while connected in the foreground.
- Synthetic Foundation tests against the actual production dispatcher,
  connection, registration store, push protocol, and push service.

The relay uses the existing Beeper provider gateway. No separate server or USB
connection is required for normal relay operation. It does **not** remove the
need to keep the iPhone online. Backgrounding the relay disconnects it; returning
to the foreground resumes its registration.

## Client compatibility

**Do not expect this relay alone to repair the unmodified stock OpenBubbles app.**
The verified repair includes matching client changes for the native account
bundle, push protocol, and Dart/Rust login-state lifetime.

The client source is on
[`agent/native-relay-client-20261020`](https://github.com/Xare123/openbubbles-app/tree/agent/native-relay-client-20261020).
The relevant immutable versions are:

| Component | Version and evidence |
| --- | --- |
| Relay source | `161f35952c22cb59b8cb3a034e176097f2df8b60`, extracted without changing runtime source |
| Live-tested Android client | `624f17ec02746e6febab5b5f5dfb77189d4908b6` |
| Subsequent client retry-state repair | `ee1ca5fee0820196e602987ccf4cff6e79adcd2c`, source/native tests and signed APK build passed; not yet installed for another Pixel check |
| rustpush | `5be3547cf0b5163568d4bb7f3b04e1b963464db2` |
| apple-private-apis | `a285eb285f03d574ce24aaa11a77931ad60db226` |

On the authorized test devices, the native relay and matching Android client
completed fresh Apple Account login and delivered a normal iMessage. A retained
session also reopened after a real process restart without another password or
verification code and delivered a further test message. This is specific live
evidence, not a claim of universal iOS compatibility or production readiness.

Fresh SIM registration, the separately restricted Android secondary-user SMS
path, and full CloudKit qualification are **not** established by those results.
The relay does not itself synchronize message history.

## Build and install

Requirements: macOS with full Xcode selected, the iPhoneOS SDK and `mig`, and an
owned iPhone capable of running this privileged app through TrollStore. The
build targets ARM64 and iOS 15 or later; that deployment target is not a claim
that every such OS/device supports the required private interfaces. Ordinary
App Store installation or normal ad-hoc sideloading does not grant these
entitlements.

```sh
git clone https://github.com/Xare123/NativeValidationRelay.git
cd NativeValidationRelay
output="$(mktemp -d)/native-relay"
OB_SOURCE_SHA="$(git rev-parse HEAD)" bash ./build.sh "$output"
```

The output directory must not already exist. The build runs the four Foundation
test executables, generates the Mach client with the installed Apple SDK,
checks finite IPC timeouts, builds ARM64, and verifies its ad-hoc signature.
It produces `OBNativeRelay.tipa`, `SHA256SUMS`, `SOURCE_SHA`, and `BUILD_TOOLS.txt`.
An ad-hoc signature check is not App Store signing, notarization, or proof of
runtime entitlement availability. Generated SDK sources remain outside Git.

For host tests alone, use a different new absolute directory:

```sh
output="$(mktemp -d)/foundation-tests"
bash ./tests/run-foundation-tests.sh "$output"
```

Install the locally built TIPA with TrollStore. Open **OB Native Relay**, leave
it in the foreground and connected, then enter its registration code in the
matching OpenBubbles client. Enter the Apple Account password and 2FA code in
the client, not the relay. Do not run competing clients against the same relay
registration while diagnosing sign-in. No jailbreak or USB controller is
included in this repository.

The manual-only GitHub workflow builds and runs credential-free source tests.
It does not contact a test account, install on a device, or publish a release.

## Security and limitations

Treat a registration code and relay ticket as credentials. Do not put them,
native proofs, device identifiers, passwords, verification codes, or personal
logs in issues or source commits. The runtime registration tuple is persisted in
the app's `NSUserDefaults`; it is **not** stored in the Keychain. The native APS
private key is accessed in the iPhone Keychain and is not exported.

Native push commands use ticket-bound authentication, replay limits, deadlines,
certificate pins, and response authentication. These retain the existing
gateway/ticket trust boundary; they do not provide new end-to-end secrecy from
the gateway. Timeouts do not make private APIs interruptible. See
[SECURITY.md](SECURITY.md) before using an account you depend on.

This is unofficial interoperability work and is not affiliated with or endorsed
by Apple, Beeper, or the upstream OpenBubbles project. Private APIs and provider
behavior can change. There is no public binary release or independent security
audit at this initial source publication.
