# Security boundaries

This experimental app uses privileged private iOS interfaces. Review its source,
entitlements, client compatibility, and operational needs before installing it.
No independent security audit is claimed.

## Data and authority

- The relay does not accept an Apple Account password, verification code, or
  message body. Those stay in the matching messaging client.
- Registration code, ticket, and gateway URL are saved together in the app's
  `NSUserDefaults`. Do not describe this storage as Keychain protection.
- The private APS key stays on the iPhone in its existing Keychain context. The
  client receives permitted signatures and public identity material, not the key.
- Device NAC, empty account qualifier, and push/registration signing are distinct
  paths. An account qualifier is not a device-validation replacement.
- The push service admits bounded discovery, fresh APNs nonce signing, and
  registration-request SHA-1 digest signing. The iPhone checks the authenticated
  envelope, nonce, operation, pin, and digest length. It does not receive and
  independently parse the full HTTP request to prove what that digest represents.
  A trusted client holding the registration code therefore has this limited
  signing authority. There is no arbitrary-code execution or private-key export.

## Transport and lifecycle

The existing Beeper gateway and the holder of a relay ticket remain trusted.
The native push MAC key is the registration code itself, not a separate secret.
Request MACs, response MACs, request IDs, certificate pins, size bounds, and replay
metadata do not establish secrecy from that gateway. Keep tickets private and
replace a compromised registration through the provider's supported controls.

TLS verification remains enabled. Native work is serialized and signing can be
quarantined after a deadline. A deadline prevents new work/replies; it does not
prove that a private native API was interrupted. Old socket/epoch completions
must not become replies on a replacement connection.

Foreground operation is required. The app disables only its own idle timer while
connected, disconnects on background entry, and can reconnect on foreground
return. It does not change the phone's system lock settings or install a screen
controller. Rebooting, disconnecting, or powering off the iPhone can prevent
operations that need fresh native material.

## Reporting problems

Do not attach real codes, tickets, account emails/passwords, signatures, device
serials/identifiers, or personal logs to public issues. Provide the source commit,
iOS/device model, matching client version, operation stage, and sanitized error
metadata. An exploitable flaw or leaked credential must not be posted publicly;
use a private reporting channel if one is available to you. This repository
does not advertise an unmonitored security email address.
