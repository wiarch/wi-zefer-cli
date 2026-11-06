# WI Arch Security Review — zefer-cli

**Review type:** source/configuration review
**Status:** initial findings; not a penetration test or independent cryptographic audit
**Reviewed branch:** main
**Canonical standard:** https://github.com/wiarch/wi-zefer/blob/main/docs/WI-ARCH-SECURITY-BASELINE.md

## Project profile

Public Node.js/TypeScript CLI for encrypting and decrypting Zefer containers. It uses Node.js crypto, Commander, Ink, supports npm and standalone binaries, and exposes an MCP server over stdio. It is a CLI companion to the upstream carrilloapps/zefer-cli and shares format compatibility with carrilloapps/zefer.

The WI Arch repository retains upstream attribution and license; this review does not imply ownership of upstream intellectual property.

## Positive controls observed

- AES-256-GCM authenticated encryption and PBKDF2-SHA256 through Node.js crypto.
- Cryptographic random generation through crypto.randomBytes.
- Async PBKDF2 rather than synchronous KDF calls.
- Hidden terminal password prompts for supported TTY modes.
- CI workflow with typecheck and smoke tests across Node.js 20 and 22.
- Security policy and threat-model documentation describe important local-client limitations.
- MCP transport is stdio by default, not an exposed network listener in the reviewed entry point.

## Findings and actions

| Priority | Finding | Required action |
|---|---|---|
| High | The public file header controls PBKDF2 iteration count. | Validate type and enforce a strict upper/lower bound before KDF execution to prevent CPU exhaustion. |
| High | Binary container lengths and framing are attacker-controlled. | Enforce bounds on header, blocks, chunks and total size; reject truncation, overflow and malformed/trailing data. |
| High | Encrypted metadata influences output behavior. | Validate metadata strictly and prevent path traversal, unsafe filenames, symlink attacks and overwrite races. |
| High | Compression is applied to attacker-controlled decrypted content. | Set decompressed-size and resource limits; test decompression-bomb cases. |
| High | MCP tools accept passphrases and filesystem paths as arguments. | Validate tool inputs, constrain path access, minimize plaintext returned, and document exposure in MCP host logs/configuration. |
| Medium | CLI passphrase flags can leak through process listings and shell history. | Prefer interactive prompts/secret manager; never log arguments; document CI-specific safe handling. |
| Medium | Attempt counter is local mutable JSON state. | Do not present it as tamper-proof; make writes atomic/concurrency-safe and use collision-resistant file identifiers. |
| Medium | CI uses third-party Actions by major-version tags. | Pin Actions to verified full commit SHAs and keep them updated. |
| Medium | CI has typecheck and smoke tests but no clearly visible dedicated security gates. | Add dependency, secret and static analysis scans with documented release-blocking thresholds. |

## CI observation

The repository workflow defines Node.js 20 and 22 jobs with typecheck and smoke tests. This review did not execute the workflow locally. A passing CI job is evidence for the checks it actually runs, not a security audit.

## Immediate review priorities

1. Harden parser/KDF parameter validation.
2. Harden metadata-to-filesystem handling.
3. Add decompression resource limits.
4. Review MCP filesystem and secret boundaries.
5. Add adversarial regression tests for all four areas.

## Limitations

This review is based on selected repository source and configuration files. It does not include runtime fuzzing, dependency audit output, a complete line-by-line review, binary reverse engineering, deployment inspection or independent cryptographic verification. Findings should be confirmed during implementation before being closed.