# Super Bonjour — Algorithm

[README](README.md) | [日本語](../ALGORITHM.md)

## 1. Bonjour discovery

Browse `_apple-mobdev2._tcp.local.` via DNS-SD and resolve each service. Collect the instance name, interface, SRV hostname and port, A/AAAA records, and TXT `identifier`, `authTag` and `authTag#N`. Parse the MAC strictly from the observed instance-name prefix (synthetic example: `02:00:00:00:00:01@…`). Do not substitute ARP or infer device identity from a hostname. Reject invalid, zero or multicast MACs. Do not infer the phone's SSID from a PC setting.

The original implementation used DNSServiceBrowse, DNSServiceResolve and DNSServiceQueryRecord. Tested integrated time windows were 15,000 ms browse, 10,000 ms resolve and 2,600 ms address queries. Make these configurable. Missing responses are not successful matches. Older devices may advertise no matching TXT information.

## 2. Local pairing records

Read HostID as a string from device pairing plists. Exclude SystemConfiguration.plist and unrelated files. The tested filename filter accepts 40 hexadecimal digits or 8 hexadecimal digits followed by a hyphen and 16 hexadecimal digits; confirm other formats before enrollment. Disable external XML DTD resolution and use a proper binary-plist parser if necessary. Do not require HostID to be a GUID: numeric strings were observed on the tested Windows installation.

## 3. Tag derivation

Based on the documented pymobiledevice3 mobdev2 technique, calculate locally. Private keys and EscrowBag are not needed for the calculation. Use the exact decoded HostID and identifier strings in UTF-8; do not trim, change case, or convert a UUID into binary GUID bytes.

```text
IKM = UTF8(HostID)
PRK = HMAC-SHA512(key = 64 zero bytes, message = IKM)
OKM = HMAC-SHA512(key = PRK, message = byte 0x01)[first 32 bytes]
expected = HMAC-SHA256(key = OKM, message = UTF8(identifier))[first 8 bytes]
received = strict_Base64_decode(TXT authTag value)
match = length(received) == 8 AND constant_time_equal(expected, received)
```

This expands HKDF-SHA512 with unspecified salt, empty info and 32-byte output. Compare each `authTag` or `authTag#N`. Treat duplicate TXT keys, missing/malformed values and inconsistent resolves as unavailable. A tag is a discovery hint; it does not cryptographically bind the advertised MAC or service name. It is not full authentication against malicious LAN participants or replay.

### Synthetic test vector

Invented inputs, unrelated to any real device:

- HostID: `SB-DEMO-HOST-001`
- identifier: `11111111-2222-4333-8444-555555555555`
- expected tag (hex): `D0C98FD99271C4C3`
- expected tag (Base64): `0MmP2ZJxxMM=`

## 4. Matching decision

```text
records = load_device_records_read_only()
require target_UDID exists exactly once
require all relevant HostIDs readable
require target HostID is not shared with any other record

candidates = []
for service in browse_and_resolve():
    if service has no usable identifier/tags:
        log unavailable or no-auth; continue
    matches = all records whose computed tag matches any valid received tag
    if target is among multiple matches: stop without writing
    if matches == [target]:
        require valid service MAC and consistent interface responses
        candidates.add(service MAC and service evidence)
require exactly one distinct, consistent target service
return its MAC
```

Deduplicate responses for the same service across interfaces. Abort on inconsistent responses for a target candidate. Missing information from unrelated devices alone need not veto a unique target match. Never write on multiple target candidates. Unobserved devices and shared HostIDs limit uniqueness; restrict a shared-HostID environment to diagnostics. A trusted session to verify UDID independently would be required there and is not implemented or validated in SB.

## 5. Update flow

```text
read live target plist and hash its bytes
find target MAC using fresh Bonjour responses
if same MAC: finish without writing
show old/new values; request explicit confirmation
close iTunes; elevate only if needed; lock against concurrent writes
re-read target; require original hash unchanged
re-run target discovery; require same MAC (UUID may legitimately change)
stop relevant Apple processes/services safely
re-read target under write lock; require unchanged
create and verify before-backup outside Lockdown
replace only the validated WiFiMACAddress field bytes
verify saved bytes and non-MAC byte identity; create verified after-backup
on write/verify failure: restore verified before-backup and verify again
restart previously stopped service; verify file again
ask before PC reboot; verify actual Wi-Fi sync in iTunes
```

References: [mobdev2 technique and limitations](https://doronz88.github.io/pymobiledevice3/latest/internals/idevice-protocol-layers/#wi-fi-recognizing-a-device-on-the-network), [RFC 5869](https://www.rfc-editor.org/rfc/rfc5869).
