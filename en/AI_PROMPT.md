# Super Bonjour — Prompt for AI-assisted implementation

[README](README.md) | [日本語](../AI_PROMPT.md)

Provide this prompt together with the specification, algorithm and test results. Do not upload real pairing files or backups.

---

I want a local utility to restore iTunes Wi-Fi synchronization between my own Windows PC and my own already trusted iPhone. Read the attached Super Bonjour documents and design an implementation. Identify a changed private MAC against an existing pairing record; update only the selected plist's WiFiMACAddress when necessary.

My environment (provide local verification instructions for missing items):

- Windows and iTunes versions: not filled in
- Bonjour installation: not filled in
- Target device and UDID: verify locally; do not publish or transmit
- OFF MAC: verify locally
- Lockdown and backup location: verify locally

First create a read-only diagnostic. Display the Bonjour service name, hostname, MAC, IP, identifier, tag availability and matching outcomes. Validate the HostID/tag computation using the synthetic vector in ALGORITHM.md. Refuse to update if HostIDs are shared, matching is unavailable, multiple target candidates exist, or responses conflict. Tags are not full authentication; restrict use to a trusted LAN. Do not access unpaired devices.

Only after diagnostics succeed, implement a separate update version with explicit approval, minimal privileges, target restriction, recheck, before/after backups, byte verification and rollback. Test the failure paths with synthetic files before modifying live pairing records. Default to no change in automatic update mode. Require iTunes to close; restore stopped services and ask before rebooting.

Do not transmit or display/log raw private keys, EscrowBag, HostID, SystemBUID, authTag, certificates or whole pairing records. Protect backups locally with restricted permissions. Redact device, network and path identifiers from published logs. Do not disable UAC, weaken ACLs, automatically install external software or create persistent elevated tasks.

Consider Windows PowerShell 5.1 and locally installed Bonjour first, and explain choices for my environment. Distinguish new implementation work from validated functionality. I will conduct the Windows real-device tests. Never claim tests you have not actually performed.

UI: use existing orange for normal text, pale cyan for acquired values only, red for errors. Do not change the console palette. Use blank lines and aligned comparison colons. Verify that presentation changes do not change functional code.

Complete and validate diagnostics first, then proceed to the update version after reviewing results.
