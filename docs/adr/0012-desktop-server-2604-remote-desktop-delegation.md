# ADR 0012: Delegate the desktop-server remote session on Ubuntu 26.04

## Status

Accepted — 2026-10-14. Supersedes the 24.04-only boundary set in ADR 0011.

## Context

ADR 0011 scoped `desktop-server` to Ubuntu 24.04 amd64 because the XRDP
composition it owns needs the Xorg GNOME session, which Ubuntu 26.04 removed.
The deployment estate now provisions new desktop-server droplets on 26.04, so a
hard failure at the remote-desktop layer would block the whole profile on a
release every other composition already supports.

The owner environment operates its own remote-desktop stack (RDS over a
headless Wayland session). That stack is deliberately not this repository's
concern: it owns endpoint identity, consent state and transport, none of which
a generic public bootstrap should reach into.

## Decision

`desktop-server` supports Ubuntu 24.04 and 26.04 on amd64. The contract records
`remote_desktop.release_delegation` mapping `26.04` to
`external-owner-overlay`. On a delegated release `remote-desktop.sh` logs the
delegation and returns success from apply, verify and the CLI entrypoint
without installing, configuring or asserting any session factory — the owner
overlay owns and proves that layer.

On 24.04 nothing changes: the XRDP/Xorg composition, loopback bind and SSH
forward remain exactly as ADR 0011 decided.

## Consequences

- `is_supported_ubuntu` already accepts 26.04, so the delegated path is the
  only code change; the hard "24.04 only" failure disappears.
- Nothing in this repository may claim the 26.04 remote session works: the
  evidence matrix marks the composition, and the owner overlay's own
  verification proves the live session.
- If a future release restores an Xorg-capable GNOME session, the delegation
  entry — not the profile's release list — is where support moves.
