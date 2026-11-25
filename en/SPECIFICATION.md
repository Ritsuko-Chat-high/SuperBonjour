# Super Bonjour — Implementation specification

[README](README.md) | [日本語](../SPECIFICATION.md)

## 1. Purpose and prerequisites

Restore Wi-Fi sync for an owned, already trusted device when the registered MAC and the Bonjour-advertised MAC differ. Do not create new pairing records, unlock devices, change privacy settings or access other people's devices. If normal iTunes settings or USB reconfiguration resolve the issue, this workaround is unnecessary.

The tested implementation used a BAT launcher, Windows PowerShell 5.1, C# and the locally installed Bonjour `dnssd.dll`. Confirm the environment and fail safely on missing dependencies or permissions. The expected pairing directory is `%ProgramData%\Apple\Lockdown`; if absent, ask the user to verify the correct path instead of creating it.

## 2. Enrollment (not implemented in the tested personal version)

| Setting | Obtain and verify |
|---|---|
| TARGET_UDID | Use a trusted local tool with the selected USB-connected device; match the corresponding existing plist and confirm with the user |
| OFF_MAC | If required, temporarily disable private addressing on that network and read the hardware MAC; the user restores their chosen setting |
| BACKUP_DIR | Private local directory outside Lockdown, with restricted access and no cloud synchronization |
| LOG_DIR | Local directory for redacted logs |

UDID is the pairing filename identifier, not the Bonjour UUID. Do not guess the target from a MAC, hostname, candidate count, or file timestamp. Stop until the target is verified.

## 3. Modes

- **Read-only diagnostic:** mandatory first implementation; display discovery, matching and MAC comparison without edits.
- **Auto detect and update:** require a unique matching target and explicit user approval; default to no change. If addresses already agree, do not write.
- **Restore OFF:** explicit manual restoration of the enrolled hardware MAC in the known target plist; discovery is unnecessary. The user verifies the corresponding phone setting.
- **Backup only:** make a verified copy of the target plist; never overwrite an existing backup.
- **Recovery:** restore the verified immediate pre-write backup on failure. For manual recovery, verify target UDID, backup contents and current pairing state. Do not blindly restore an obsolete pairing record.

## 4. Update safeguards

1. Read all relevant pairing HostIDs and ensure the target is unique as specified in ALGORITHM. Other records are read-only.
2. Close iTunes. Elevate only when necessary and after explicit approval; never disable UAC, weaken ACLs or register persistent elevated tasks. Revalidate settings, target, original file hash and candidate MAC after elevation.
3. Prevent concurrent writes (the tested version used a named mutex). Safely stop relevant Apple processes and services and open the target in a mode that disallows other writers. The tested implementation stopped Apple Mobile Device Service and AppleMobileDeviceProcess. Explain the impact to the user; abort if they cannot be stopped.
4. Verify that the target is a regular file, not a symbolic link or junction, and matches the enrolled filename. Write a new backup outside Lockdown, flush, reread and verify exact byte equality.
5. Parse the plist and replace only the unique top-level `WiFiMACAddress` string. The tested file was XML with a 17-character colon-separated MAC. Preserve encoding, field length, file length and every other byte. Abort if the offset cannot be determined safely. Do not enable binary-plist writes until a dedicated parser and synthetic-file tests are validated.
6. Reread the modified file, parse the new MAC, compare all non-MAC bytes, and save and verify a separate after-backup. If writing or verification fails, restore the immediate verified before-backup and verify again. If rollback fails, report it and stop.
7. Restart previously stopped services in a finally-style cleanup and verify the resulting file again. Report if the file changes after services restart; do not silently keep rewriting it. Ask before rebooting. Confirm actual iTunes Wi-Fi sync separately.

## 5. UI and logging

Use existing orange for labels, titles, messages, defaults and choices; pale cyan only for runtime-acquired values; red for errors. Do not change the user's console palette. Align comparison colons and separate groups with blank lines. Keep presentation edits isolated from discovery, matching and writing.

Log candidate counts, match outcomes, old/new MAC comparison, recheck, backup, verification and rollback results. Distinguish no response, missing fields, mismatch, ambiguity and errors. Never log secret values. Redact MACs, UDIDs, IPs, UUIDs, SSIDs, names, paths and certificate fingerprints before publishing logs. Treat full backups as confidential pairing material.

## 6. Acceptance criteria

Other plists remain unchanged; all bytes outside the target MAC field remain unchanged; both backups verify; no unapproved or ambiguous writes occur. Confirm actual sync on a real device. Do not report untested environments as supported.
