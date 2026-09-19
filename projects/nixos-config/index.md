---
title: "NixOS Config"
author: "openai-codex/gpt-5.6-sol"
layout: project.njk
description: "One flake for three NixOS machines and two OpenWrt images."
thumbnail: ""
date: 2026-08-26
status: "active"
featured: false
tags: ["linux", "nix", "infrastructure"]
media: []
---

# NixOS Config

I keep all of my system configuration in one Nix flake. If I break a machine, I
want the fix in git instead of in a shell history I will never find again. The
repo currently covers three NixOS machines and two OpenWrt images:

- `gs-thinkpad-t480s`, my daily-use laptop;
- `gs-server`, including a Windows VM with GPU passthrough;
- `gs-pi4`, the media and backup server;
- `gs-openwrt-one`, a declarative WISP/router image; and
- `gs-pi1-parents`, an OpenWrt image for a remote Tailscale exit node and subnet
  router.

There is also a QEMU target for testing the Pi configuration, but that isn't a
real machine.

## Shared modules

Each NixOS machine has a small host file. Common settings live in shared
modules, each host picks a role, and the role pulls in features such as Sway,
audio, containers, or laptop power handling. Hardware-specific workarounds stay
with the machine that needs them. Home Manager configures the programs I use on
the laptop and server from the same repo.

I keep decision-making shell scripts outside the Nix expressions so I can feed
them fake state in tests. That has caught mistakes in lid handling, display
layout, battery warnings, boot ordering, media checks, and OpenWrt route setup
without making the real machine the test fixture.

Secrets use sops-nix. NixOS hosts decrypt with an age key derived from their SSH
host key. The OpenWrt builds inject their first-boot credentials while building
the image, so none of the plaintext ends up in git.

## Deploying the Pi

The ThinkPad builds and deploys `gs-pi4`. Nix evaluates the system for
`aarch64-linux`, then runs native aarch64 builders through QEMU/binfmt. It is
slow native-platform emulation, not a cross-compiled NixOS system. The completed
store paths are signed, copied to the Pi, and activated over SSH.

The Pi runs my media stack and stores btrfs snapshots from the laptop. Media,
service state, and backups have separate subvolumes. The free-space guards read
the media qgroup because `df` can show plenty of free space after that quota is
already full. I can boot the Pi configuration in QEMU before deploying it, and
the service checks cover the live path from a media request to the file showing
up in Jellyfin.

## OpenWrt from the same repository

Neither OpenWrt target is a NixOS host. The OpenWrt One image configures a Wi-Fi
uplink, wired LAN, local access point, Tailscale, and DNS forwarding. The
Raspberry Pi 1 image uses OpenWrt because nixpkgs has no ARMv6 binary cache.
OpenWrt's ImageBuilder can assemble its roughly 21 MB image without spending
weeks compiling a NixOS toolchain on a 700 MHz Pi.

The repository is private and lives at `/etc/nixos`.
