---
title: "NixOS Config"
layout: project.njk
description: "Flake-based NixOS configuration managing a ThinkPad, a Pi 4 media server, a Framework server, and an OpenWrt router from a single repo."
thumbnail: ""
date: 2026-08-26
status: "active"
featured: false
tags: ["linux", "nix", "infrastructure"]
media: []
---

# NixOS Config

## Overview

This is my NixOS configuration: a single flake-based repository that manages
every machine I run. The repo has about 5,400 lines of Nix across 78 files,
with 491 commits since January 2026.

The git history tells the whole story of learning NixOS from scratch. Early
commits are trial-and-error ("Try fixing STM32CubeMX to use the fake home",
"Hopefully fix bug that just locks up the whole computer"). Later commits are
surgical ("waybar: clock via glibc date, libstdc++ tzdb reads Vancouver an
hour behind"). The repo grew from a bare GNOME desktop into a fully declarative
sway setup, a multi-host infrastructure, and a tested shell script collection.

The hosts are:

- **gs-thinkpad-t480s** (primary daily driver, Intel i5-8250U)
- **gs-server** (Framework-class server with a Windows 11 VM via libvirt/VFIO
  passthrough)
- **gs-pi4** (Raspberry Pi 4, media server, cross-compiled via QEMU binfmt
  emulation)
- **gs-openwrt-one** (OpenWrt router, built declaratively from Nix)
- **gs-pi4-vm** (QEMU test VM for the Pi 4 config)

## Architecture

The flake entry point wires each host together with home-manager as a NixOS
module. The module tree has four layers:

1. **Core** (`modules/core/`): packages and settings that apply to every host.
2. **Roles** (`modules/roles/`): one per host kind (laptop, server). A host
   imports exactly one role.
3. **Features** (`modules/features/`): 22 opt-in modules covering audio, btrfs,
   fonts, desktop, sway, virtualization, Steam, Flipper Zero, Firefox,
   containers (distrobox), and more. A role selects which features it needs.
4. **Hardware** (`modules/hardware/`): per-machine quirks (ThinkPad lid
   behavior, Thunderbolt dock handling, PCIe wakeup management).

Home-manager configs live in `home/dotfiles/`, one file per program: sway
(274 lines), waybar (355 lines), kitty (202 lines), helix, kanshi, battery
management, fuzzel, darkman, swayidle, and more.

## Testing

Shell scripts that make runtime decisions are split into standalone files with
fixture-based test suites. There are 11 test scripts covering:

- **Lid decision:** tests against synthetic `/proc/acpi`, `/sys/bus/thunderbolt`,
  and `/sys/class/drm` trees. Every combination of docked/undocked, AC/battery,
  and lid open/closed has a fixture.
- **Thunderbolt state:** tests against synthetic PCI device trees to detect
  whether the controller is wedged.
- **Display plan:** tests against fake `swaymsg -t get_outputs` JSON to verify
  layout geometry for extend, mirror, swap, and external-only modes.
- **Workspace plan:** tests against fake `swaymsg -t get_workspaces` JSON to
  verify workspace reassignment across all three modes.
- **Battery level:** tests the threshold transitions and notification
  deduplication.
- **Waybar formatting, AV stepping, GPU busy detection, secrets guard,
  snapper orphan cleanup.**

A `Makefile` target runs the full suite. Every script takes overridable paths
(e.g. `LID_STATE`, `TB_DEVICES`, `DRM_DIR`) so tests substitute fixture trees.

## ThinkPad Hardware Management

### Lid and Power

The lid handler reads four inputs: lid state, Thunderbolt authorization,
connected displays, and AC power. It outputs one of four actions:
`none`, `stay-awake`, `suspend`, or `suspend-then-hibernate`.

The decision tree: a connected external display or an authorized Thunderbolt
device means "docked" (stay awake). Otherwise, AC means suspend, and battery
means suspend-then-hibernate (S3 first, S4 after 30 minutes via RTC wake).

On resume, the system re-arms PCIe root port wakeup, checks the lid state, and
if the lid is still closed (e.g. a hibernate self-wake), goes straight back to
sleep. Otherwise it restarts DNS and Wi-Fi and fires Thunderbolt recovery.

### Thunderbolt Recovery

The T480s has an Alpine Ridge LP Thunderbolt controller that wedges after
hibernate. A hung ICM (internal connection manager) leaves the PCI bridge
functions alive but the NHI (native host interface) gone, or the NHI present
with an empty domain. Either way, hotplug stops working.

I wrote a detection script that checks the PCI topology for this specific
failure mode, and a recovery script that hard-resets the chip: it removes the
PCI functions, cuts `force_power` for a 10-second off dwell (2 seconds was not
enough), rescans, and releases. This runs automatically on resume as a
systemd service.

### Dock USB

The Lenovo USB-C dock (40AY) had SuperSpeed link failures. I added a udev rule
to hold the Alpine Ridge xHCI controller at `power/control=on` when the dock
is connected, which prevents the controller from power-gating and dropping
the link.

## Desktop (Sway)

The desktop evolved from a GNOME setup through a long swaybar/i3blocks phase
into the current sway-native stack. Every component is declared in Nix.

### Waybar

Waybar replaced swaybar/i3blocks after I hit limits with SNI tray icon
rendering. I wrote the config from scratch: pill-shaped blocks, Nightfox color
palette, function-matched colors (green accent for healthy, yellow for battery
warnings). The clock uses glibc `date` instead of the built-in widget because
`libstdc++` reads the Vancouver timezone an hour behind.

### Display and Workspace Management

- **kanshi** manages output profiles with wildcard external matching. The
  external monitor is discovered at runtime, not hardcoded by name.
- **display-plan.sh** computes the sway output layout from connected displays.
  It handles extend, external-only, mirror, and swap modes, with automatic
  scale detection (4K panels get 2x) and subpixel assertion.
- **workspace-plan.sh** reassigns workspaces across outputs. Odd workspaces go
  to the internal panel, even to the external. A `swap` keybind flips sides
  live.

### Other Desktop

- **Custom xkb keymap** remaps Caps Lock to Escape and Left Alt to
  Hyper_L/Mod3.
- **fuzzel** as the application launcher (font matched to kitty at size 12,
  scaled per output).
- **darkman** auto-switches GTK theme and wallpaper between light and dark
  based on solar position (latitude/longitude of Vancouver).
- **Wallpaper rotation** renders wallpapers from source SVGs to 4K PNGs at
  build time, with an optional NixOS emblem watermark composited via
  ImageMagick.
- **swayidle** locks with swaylock-effects (blurred, dimmed background) after
  30 minutes, then turns off displays.
- **AV stepping** rounds volume and brightness percentages to the next step
  boundary (tested).
- **dnscrypt-proxy** routes DNS over HTTPS to Cloudflare.

### Battery

A systemd user timer polls battery state and fires notifications at low and
critical thresholds. The threshold transitions are deduplicated (no repeated
warnings) and tested against fixtures. At the critical level, the system
auto-hibernates.

## Server (GPU Passthrough)

The Framework server runs a Windows 11 VM via libvirt with VFIO GPU
passthrough. IOMMU is enabled with `iommu=pt`, and the RX 580 detaches from
`amdgpu` on demand via `virsh nodedev-detach`. The host keeps ROCm/OpenCL for
compute when the GPU is not passed through. The VM definition is declarative
(NixVirt), but `active = null` so a NixOS rebuild never disturbs a running
session.

The VM sits on a libvirt NAT bridge (not macvtap, because Wi-Fi stations
cannot present a guest's MAC to the AP). A qemu hook script manages iptables
port-forwarding rules so the guest is reachable from the LAN. The forwarding
plan is generated by a tested script (`win11-forward.sh`) and applied/torn
down on VM start/stop.

## Pi 4 Media Server

The Pi 4 runs a media stack (Jellyfin, Sonarr, Radarr) on btrfs with
subvolume quotas. The config cross-compiles on the T480s via QEMU binfmt
emulation and deploys over the network. A test VM (`gs-pi4-vm`) boots the
full Pi config in QEMU with a Makefile that builds, launches, prints service
URLs, and offers SSH access, so I can validate service changes before
deploying to hardware.

Specific work:

- **btrfs subvolume management.** Media and state are separate subvolumes,
  with a 256 GiB quota on media. Boot-time assertions verify the subvolumes
  exist and the quota is applied.
- **Multi-device btrfs boot race.** `/srv/media` automounts to survive the
  race where the second device in a multi-device btrfs filesystem is not yet
  scanned at mount time.
- **Snapper orphan cleanup.** A tested script reaps orphaned snapper snapshots,
  with a sleep inhibitor to prevent the machine from suspending mid-cleanup.
- **Media management suites.** A cleanup dashboard, season pack planner, health
  checker, and a background pause system for Jellyfin that suspends transcoding
  when the client disconnects.
- **Moonfin/Delfin.** I packaged the Moonfin Jellyfin client with its native
  library dependencies (libepoxy, libXv, libmpv) and a desktop entry so it
  appears in fuzzel.

## OpenWrt Router

The OpenWrt One runs in WISP mode: a Wi-Fi client uplink on 5 GHz through NAT
to wired LAN plus a local AP, with hardware flow offloading. The firmware
image is built declaratively from Nix using `nix-openwrt-imagebuilder`.

Features:

- **Tailscale:** Joins the tailnet on first boot via a uci-defaults script.
- **DNS:** Routes LAN DNS through the Pi's AdGuard Home for ad blocking, with
  a reservation fallback.
- **2.5G port:** Folded into `br-lan` for the downstream switch.
- **Secrets:** Wi-Fi credentials and Tailscale auth key encrypted with sops-nix.
- **Package index patching:** OpenWrt rebuilds its package indexes in place, so
  every cached `packages.adb` hash goes stale on point releases. I maintain a
  `freshHashes` table that patches the fixed-output derivation hashes at build
  time.

The move from OpenWrt 25.12 replaced `opkg` with `apk`, which required
reworking the package installation flow. Flash and update go through Makefile
targets.

## Firefox

Firefox extensions and policies are declared in Nix. Extensions
(uBlock Origin, Bitwarden, Vimium, Privacy Badger, ClearURLs, Facebook
Container, YouTube Shorts Block, Adaptive Tab Bar Colour) are
`force_installed` from AMO's latest URL and track upstream automatically.
Telemetry, Pocket, and sponsored content are disabled via policy.

## Claude Code Integration

The repo manages Claude Code's own configuration (`claude.nix` generates
`~/.claude/settings.json`). I also wrote custom Claude skills as Nix modules:
a `/workflow` skill with three modes (auto, user, claude), a `/debug`
root-cause skill, a returns-processing skill (covering Amazon and DigiKey),
and a media-grab fix skill. A `sudo` workaround uses fuzzel as an askpass
agent so that Claude's ttyless `sudo` calls pop a GUI prompt on the desktop
instead of hanging.

## Secrets

Managed with sops-nix. Each host has an age key derived from its SSH host key.
Encrypted secrets live in `secrets/secrets.yaml` and decrypt to `/run/secrets/`
at activation.

## Repository

Private repository at `/etc/nixos`.
