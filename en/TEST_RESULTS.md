# Super Bonjour — Anonymized real-device results

[README](README.md) | [日本語](../TEST_RESULTS.md)

## 1. Environment

Windows 10, iTunes, Bonjour, iPhone 12 Pro Max, multiple Apple devices on one LAN. The target pairing file was XML. Four device pairing records had distinct HostIDs. The target contained nine fields: HostID, SystemBUID, WiFiMACAddress, three certificates, two private keys and EscrowBag. No secret values are published.

## 2. Read-only diagnostics (2026-11-25)

Labels A/B/C/D below stand for anonymized values, not literal addresses. Identical labels within a column represent identical values. All four UUIDs differed.

| Condition | SSID | MAC | IPv4 | UUID | Target HostID | Other 3 records | Candidate count |
|---|---|---|---|---|---|---|---:|
| Off | A | A | A | A | Match | No match | 3 |
| Fixed | A | B | B | B | Match | No match | 3 |
| Rotating | A | B | B | C | Match | No match | 3 |
| Other SSID, rotating | B | C | C | D | Match | No match | 2 |

All four runs obtained a tag matching only the target HostID. The original OFF MAC remained in the pairing file during the fixed and rotating identification tests. Both SSIDs connected to the same LAN. No fixed SSID, MAC, IP or UUID was required for matching. The displayed SSID in the other-SSID diagnostic log was a static program setting; the actual SSID was confirmed on the phone.

A separate earlier scan did not detect the target. Some older devices provided no identifier or tag, and one device visible in iTunes was not detected by the diagnostic. The table shows four successful conditions, not a claim of a perfect scan success rate.

## 3. Integrated version tests

| Version | Result |
|---|---|
| v1.6 | Backup succeeded; authentication condition aborted before writing |
| v1.61 | Restored known OFF MAC; Wi-Fi sync succeeded after PC reboot |
| v1.7 | Target matched; incomplete SRV/TXT for another device caused an unnecessary abort |
| v1.72 | After changing unrelated unavailable responses to logging only: target recheck, before/after backups, MAC-only modification, byte verification, PC reboot, iTunes detection and actual sync succeeded (user report) |
| v1.77 series | Identification and no-write-on-equal-MAC observed after UI adjustments; full regression testing not established |

## 4. Not yet validated

- Natural long-term rotation and recovery; operating system and iTunes updates.
- Other PCs, models, Apple Devices, installation types, shared HostIDs and binary-plist writes.
- Recovery from interruption, disk errors, insufficient permissions or damaged backups.
- Complete Bonjour discovery coverage, why older devices omitted tags, and the internal cause of the iPhone detection failure in iTunes.

These are not claimed successes. Raw logs and real MACs, IPs, SSIDs, UDIDs, UUIDs, HostIDs, names, private keys, certificate fingerprints and local paths are omitted.
