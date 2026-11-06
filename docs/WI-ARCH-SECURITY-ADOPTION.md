# WI Arch Security Adoption — zefer-cli

**Status:** repository-level adoption record; not a certification or independent audit.
**Canonical baseline:** https://github.com/wiarch/wi-zefer/blob/main/docs/WI-ARCH-SECURITY-BASELINE.md
**Stack:** Node.js 20+, TypeScript, Commander, Ink, Node.js crypto, MCP over stdio.

## 1. Scope and responsibility

zefer-cli is the terminal companion to the upstream Zefer application and must preserve the shared .zefer file format. This repository adopts the WI Arch baseline for its CLI, cryptographic parser, filesystem access, npm distribution, standalone binaries and MCP server.

The upstream project remains the authority for shared format compatibility. WI Arch security requirements are additional safeguards, not a claim that the upstream project has adopted them.

## 2. Applicable controls

| Control | CLI implementation requirement |
|---|---|
| Secret handling | Prefer hidden interactive prompts or an approved secret-store integration. Warn that command-line flags can appear in shell history, process listings, CI logs and MCP client configuration. Never log secret arguments. |
| Input validation | Treat every .zefer file as untrusted. Bound header length, KDF iterations, chunk lengths/counts, total input size and decompressed output before expensive work or allocation. |
| Cryptography | Use Node.js platform crypto APIs, CSPRNG salts/IVs, authenticated encryption and async PBKDF2. Do not invent algorithms or silently change format parameters. |
| Filesystem | Use safe path handling, restrictive permissions for sensitive state, atomic writes where supported, and explicit overwrite confirmation. Prevent path traversal from encrypted metadata such as file names. |
| Output | Do not print decrypted secrets unless the user explicitly chooses stdout. Avoid secret values in diagnostics, progress output, exceptions and telemetry. |
| MCP | Treat MCP tool arguments and file paths as untrusted. Validate schemas and paths, minimize returned plaintext, require explicit user intent for writes, and do not expose network transports without a separate review. |
| Supply chain | Commit lockfiles, use npm ci, keep dependencies updated, scan dependencies and secrets, pin GitHub Actions to full SHAs for hardened workflows, and produce verifiable release artifacts. |
| CI/release | Require typecheck, tests, build and security checks on supported Node versions. Release only from reviewed tags; publish checksums and provenance where feasible. |
| Vulnerability handling | Keep private disclosure instructions current, triage cryptographic/parser issues urgently, and coordinate shared-format vulnerabilities with upstream Zefer maintainers. |

## 3. Current repository evidence

- AES-256-GCM and PBKDF2-SHA256 are implemented with Node.js crypto.
- Salts, IVs and generated keys use crypto.randomBytes.
- PBKDF2 is asynchronous.
- Password prompts have TTY-specific hidden-input paths.
- .zefer files are cross-compatible with the upstream browser application.
- The package exposes an MCP stdio server.
- CI runs typecheck and smoke tests on Node.js 20 and 22.
- SECURITY.md documents command-line argument exposure, local attempt-counter limitations, clock manipulation, memory inspection and compromised-host limitations.

These observations are source/configuration evidence only; they do not establish deployed security or independent cryptographic assurance.

## 4. Required hardening backlog

The following items require code-level work and tests. Do not mark them complete based on this document.

1. Validate the public header schema and enforce a narrow supported PBKDF2 iteration range before invoking PBKDF2. Reject non-integers, zero, negative, excessive and unsupported values.
2. Add strict bounds and structural validation for header size, main-block size, chunk framing, chunk count, truncation, trailing bytes and total file size.
3. Validate decrypted metadata fields and types before using them for paths, decompression, output naming or policy checks.
4. Ensure decompression has output-size and resource limits to prevent decompression bombs.
5. Audit all file output paths, including names recovered from encrypted metadata, for traversal, symlinks, overwrite races and unsafe permissions.
6. Review MCP tools for arbitrary path access, excessive plaintext returned to clients, argument validation, overwrite behavior and tool descriptions that encourage putting secrets in persistent config.
7. Add tests for malformed files, extreme KDF values, oversized headers/chunks, truncated data, malformed metadata, decompression limits, path traversal and MCP misuse.
8. Review attempt tracking: define its threat boundary, use collision-resistant per-file identifiers, handle concurrent writes safely and avoid implying it is tamper-proof.
9. Improve secret-input guidance for CI and MCP. Environment variables are not universally safe; prefer the host's secret manager or interactive prompt and avoid command-line arguments.
10. Add security scanning and release controls with explicit blocking thresholds; pin third-party GitHub Actions to full commit SHAs after verifying revisions.

## 5. Release gate

A release must not proceed with an unresolved critical issue or a high-severity exploitable issue in cryptography, parser, filesystem writes, secret exposure or MCP access. Exceptions require a named owner, documented rationale, compensating control and expiry date.

## 6. What this record does not claim

- No penetration test or independent cryptographic audit has been performed.
- Local attempt counters, expiration and IP metadata are not strong server-enforced access controls.
- A local CLI cannot protect plaintext from a compromised operating system, malicious process, keylogger or privileged user.
- Encrypting a file does not secure its source copy, decrypted output, shell history, terminal scrollback, backups or MCP client logs.
- Compatibility with upstream Zefer does not imply identical security posture.