# CompTech Cyber Intrusion Investigation: A Forensic Analysis

**Course:** CSCI 6637 — Introduction to Cyber Forensics, University of New Haven
**Instructor:** Prof. Murat Gunestas
**Date:** May 2026
**Domain:** Digital Forensics & Incident Response (DFIR)

## Overview
A full forensic investigation into a simulated cyber intrusion at "CompTech," a company that
reported unauthorized access to its internal Windows Server 2019 infrastructure and the
exfiltration of its patent and project MySQL database. The investigation was conducted against
forensic disk images and network packet captures from two machines — a suspect employee's
workstation and the victim server — with the goal of confirming whether unauthorized access
occurred, whether data was exfiltrated, and reconstructing the full attack chain.

## Tools & Technologies
- **Autopsy 4.22.1** — disk image analysis (two isolated cases, one per machine, to avoid
  cross-contamination of artifacts)
- **Wireshark** — network packet capture analysis (HTTP payload decoding, TCP stream
  reconstruction, conversation statistics)
- **Volatility 3** — attempted memory forensics (see Challenges below)

## Methodology
- Forensic images were hash-verified (MD5/SHA-256) before and after analysis; all work was
  performed on forensic copies only, never original evidence.
- Two separate Autopsy cases were built with Recent Activity, Hash Lookup, File Type ID,
  Extension Mismatch Detection, Keyword Search, Email Parser, Encryption Detection, and
  Interesting Files ingest modules enabled.
- Network captures were analyzed in read-only mode using a structured filter approach:
  reconnaissance traffic isolated first (SYN-only filters), then HTTP request enumeration,
  authentication payload inspection, and web shell command reconstruction.
- All findings were logged with source artifact path, volume/inode reference, MAC timestamps,
  and hash values, then arranged chronologically for timeline reconstruction and cross-referenced
  between disk and network evidence to establish corroboration.

## Key Findings
The investigation confirmed a premeditated, multi-stage intrusion:
1. **Reconnaissance** — attacker downloaded Metasploit, WPScan, Nmap, and other tooling onto
   their workstation starting several days before the breach.
2. **Credential theft via phishing** — a cloned WordPress login page and a credential-harvesting
   script were built and the phishing link delivered over an internal chat client to a
   privileged CMS user, whose credentials were captured.
3. **Exploitation** — the stolen credentials were used to research and deploy a known
   authenticated Remote Code Execution exploit (Exploit-DB #50093) against a vulnerable
   WordPress plugin.
4. **Web shell deployment** — a PHP web shell was uploaded through the exploited plugin and
   used to read and exfiltrate the site's config file, exposing the shared MySQL database
   credentials.
5. **Persistence & full compromise** — an SSH key pair was staged for backdoor persistence, and
   a Meterpreter reverse-TCP payload was pushed to the server via a PowerShell download
   command, establishing an encrypted reverse shell session consistent with full database
   exfiltration.

Every step was corroborated across at least two independent artifact sources (disk + network),
and a full minute-by-minute timeline was reconstructed from browser history, file metadata,
PowerShell console history, chat logs, and packet captures.

## Challenges & How They Were Solved
- **High volume of unrelated browsing activity** — resolved by date-filtering to the
  investigation window and cross-referencing URLs against known attack tooling.
- **Double URL-encoded stolen credentials** — required two rounds of manual decoding, verified
  against the plaintext form payload captured in the packet trace.
- **Garbled chat database rendering** — resolved by switching Autopsy's viewer from raw text to
  its structured application view.
- **Volatility 3 memory analysis failed** due to profile/plugin incompatibility with the capture
  format. Rather than leaving a gap, process- and network-level evidence that memory analysis
  would have shown was reconstructed from disk artifacts (console history, prefetch execution
  records, key-generation timestamps) and cross-corroborated with the packet captures —
  documented as direct inference from confirmed artifacts, not speculation.

## Recommendations Delivered
Immediate credential rotation and removal of the planted web shell/payload; patching or removal
of the vulnerable plugin; an audit and purge of SSH authorized keys; a full database log audit to
scope the breach; and preventive measures including a Web Application Firewall, phishing-
focused security awareness training, chat-link monitoring, and outbound egress filtering.

## Skills Demonstrated
Disk forensics (Autopsy), network forensics (Wireshark), chain-of-custody and evidence
handling, timeline reconstruction from multi-source artifacts, exploit/attack-chain analysis,
incident report writing, and working around a tooling failure (Volatility) by substituting an
evidence-based reconstruction rather than leaving the analysis incomplete.

---
*Final case project for CSCI 6637 (Introduction to Cyber Forensics) at the University of New
Haven. CompTech, and all individuals named in the case, are part of a simulated case study for
coursework.*
