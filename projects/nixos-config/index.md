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

The Nix flake that manages every machine I actually run. The point is that if I break one, the fix belongs in git, not in a shell history I'll never find again.

It currently covers three NixOS machines and two OpenWrt image builds:

- `gs-thinkpad-t480s`, my daily-driver laptop.
- `gs-server`, a Framework-class server with a Windows 11 VM via VFIO GPU passthrough.
- `gs-pi4`, the Raspberry Pi 4 that runs the media stack.
- `gs-openwrt-one`, a declaratively-built WISP/router image for the OpenWrt One.
- `gs-pi1-parents`, an OpenWrt image for a Raspberry Pi 1 B+ that lives at a family member's house as a Tailscale exit node / subnet router.

There's also a QEMU test target for the Pi config, but it's a test target, not a machine.

## Layout

Each host has a small entry file. Common config lives in shared modules, each host picks a role (`laptop`, `server`, `pi`), and the role pulls in the features that host actually needs — Sway, audio, containers, laptop power handling, virtualization, and so on. Machine-specific quirks (T480s lid handling, Thunderbolt dock, PCIe wakeup) live in `modules/hardware/`. Home Manager configures the programs I use on both the laptop and the server from the same repo.

Anything that's a runtime decision — lid state, display layout, workspace assignment, battery transitions, waybar formatting, media health, Thunderbolt state, OpenWrt route selection — lives in a shell script outside the Nix strings, with a fixture-based test suite next to it. That's what catches boot-ordering regressions and lid-handler mistakes before they get near real hardware. Secrets are sops-nix; each NixOS host derives its age identity from its SSH host key.

## Deploying the Pi

The ThinkPad is where `gs-pi4` gets built. Nix evaluates the system as `aarch64-linux` and runs the aarch64 builders under QEMU binfmt emulation — native aarch64 builds on x86 hardware, not a NixOS cross-compile. The signed store paths are then pushed across the network to the Pi.

`gs-pi4` runs the media stack on btrfs. Media, service state, and the T480s home backups sit on separate subvolumes with quota-aware guards that read the btrfs qgroup instead of `df` — that's how I caught the case where `df` claimed 151 GiB free while the qgroup was already full and every fetcher was hitting EDQUOT. A QEMU test VM boots the full Pi config on my laptop so activation failures show up before deploying to hardware.

## OpenWrt

Neither OpenWrt target is a NixOS host. The OpenWrt One is a WISP-mode router: Wi-Fi client uplink, wired LAN, local AP, Tailscale, and DNS forwarded through the Pi's AdGuard Home. The Pi 1 B+ is deliberately OpenWrt because nixpkgs has no ARMv6 binary cache — NixOS would end up compiling glibc, systemd, and the kernel from source on a 700 MHz single core. OpenWrt's ImageBuilder assembles a ~21 MB image in seconds.

Repository is private and lives at `/etc/nixos`.
