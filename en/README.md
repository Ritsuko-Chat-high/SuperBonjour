# Super Bonjour (SB)

**A documented workaround for iTunes Wi-Fi sync on Windows**

[日本語](../README.md) | Publication candidate 0.2 / 2026-11-25

## Overview

In one Windows iTunes installation, an already paired iPhone stopped appearing for Wi-Fi synchronization after its private Wi-Fi MAC address changed. Super Bonjour associates the current address advertised by Bonjour with an existing local pairing record using its HostID and the advertised authTag. Updating only `WiFiMACAddress` in the selected plist restored Wi-Fi synchronization in the tested environment.

This is an independently tested workaround, not an official Apple fix. The internal cause of the iTunes failure is unknown. The goal is to retain the private Wi-Fi address feature rather than permanently disable it.

## Reading order

| Document | Contents |
|---|---|
| [SPECIFICATION.md](SPECIFICATION.md) | Requirements, enrollment, updating and recovery |
| [ALGORITHM.md](ALGORITHM.md) | Discovery, tag derivation, decision rules and pseudocode |
| [TEST_RESULTS.md](TEST_RESULTS.md) | Anonymized real-device results and limitations |
| [AI_PROMPT.md](AI_PROMPT.md) | Instructions for creating a personal implementation with AI |

## Scope and limitations

- Tested with Windows 10, iTunes, Bonjour and an iPhone 12 Pro Max already trusted and configured for Wi-Fi sync.
- Four successful identification conditions and an end-to-end v1.72 recovery were observed where the four pairing records had distinct HostIDs. Long-term natural address rotation, other computers, operating systems and Apple Devices are not validated.
- A shared HostID cannot distinguish multiple paired devices using this method alone. Do not automatically update in that situation.
- An authTag is a discovery hint, not full cryptographic device authentication. Use only on a trusted LAN with devices you own or administer.

## How to use these documents

This repository contains documentation only: no BAT, EXE, DLL, original plist, backup or raw log. Give the specification, algorithm, results and AI prompt to an AI assistant to design an implementation for your own environment. Review the generated code and begin with read-only diagnostics. Never send a pairing record or private key to an AI, public repository or external website.

## Related prior work (checked 2026-11-25)

- [netmuxd Issue #56](https://github.com/jkcoxson/netmuxd/issues/56): reports a mismatch between Bonjour's advertised MAC and the pairing record on Linux and recovery after updating the MAC. The proposed update script requires USB.
- [AirBattery PR #210](https://github.com/lihaoyun6/AirBattery/pull/210): implements a related MAC update on macOS. The PR describes updating again after rotation on the next USB connection.
- [GeoPort](https://github.com/davesc63/GeoPort): a separate project for iOS location simulation.

SB's contribution is a documented demonstration in one environment: combining the existing mobdev2 matching technique with a pairing-record MAC update to restore Windows iTunes Wi-Fi sync, retaining the existing pairing and rotating private-address setting without reconnecting USB. No claim of being the first worldwide or of the absence of other solutions is made.

## References and credit

- [pymobiledevice3: Wi-Fi device recognition](https://doronz88.github.io/pymobiledevice3/latest/internals/idevice-protocol-layers/#wi-fi-recognizing-a-device-on-the-network) — source for the mobdev2 tag technique; SB does not claim to have invented it.
- [pymobiledevice3 source repository](https://github.com/doronz88/pymobiledevice3) — implementation reference; review its license before copying code.
- [Apple: Private Wi-Fi addresses](https://support.apple.com/en-us/102509) — feature behavior.
- [RFC 5869: HKDF](https://www.rfc-editor.org/rfc/rfc5869) — key derivation standard.

No redistribution license is granted by this draft; the publisher must choose a license before publication. This work is independent of Apple and the referenced projects.
