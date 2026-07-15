# Security Policy

## Supported versions

This project is under active development. Security fixes target the **latest**
release and the current `main` branch. Older tags are not guaranteed to receive
patches.

| Version        | Supported |
|----------------|-----------|
| `latest` / `main` | ✅ |
| older tags     | ❌ |

## Reporting a vulnerability

Please report security issues **privately** — do **not** open a public issue,
pull request or discussion that reveals the details.

Preferred channels:

1. **GitHub Security Advisories** — use *Security → Report a vulnerability* on
   this repository to open a private advisory.
2. Otherwise, contact a maintainer privately on GitHub
   ([@giosci1994](https://github.com/giosci1994)).

Please include:

- a description of the issue and its impact;
- steps to reproduce (proof of concept if possible);
- affected version / commit and environment (OS, Docker version).

## What to expect

- We aim to acknowledge a valid report within a reasonable time.
- We'll investigate, keep you updated, and coordinate a fix and disclosure.
- Please practice **responsible disclosure**: give us a chance to release a fix
  before making details public.

## Hardening notes (for operators)

- Never commit real secrets — keep MQTT credentials and keys in `.env` (which is
  git-ignored) or pass them at runtime.
- Run the server behind an **HTTPS reverse proxy** and restrict access to the
  camera stream and dashboard on untrusted networks.

Thank you for helping keep the project and its users safe. 🛡️
