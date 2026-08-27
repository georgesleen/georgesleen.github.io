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

This is my NixOS configuration: a single flake-based repository that manages
every machine I run. About 5,400 lines of Nix across 78 files, with 491
commits since January 2026.

The git history is the real story of learning NixOS from scratch. Early commits
are trial-and-error ("Try fixing STM32CubeMX to use the fake home", "Hopefully
fix bug that just locks up the whole computer"). Later commits are surgical
("waybar: clock via glibc date, libstdc++ tzdb reads Vancouver an hour behind").

The hosts:

- **gs-thinkpad-t480s** (primary daily driver, Intel i5-8250U)
- **gs-server** (Framework-class server with a Windows 11 VM via VFIO
  passthrough)
- **gs-pi4** (Raspberry Pi 4, media server, cross-compiled via QEMU binfmt)
- **gs-openwrt-one** (OpenWrt router, built declaratively from Nix)
- **gs-pi4-vm** (QEMU test VM for the Pi 4 config)

## Architecture

The flake entry point wires each host together with home-manager as a NixOS
module. The module tree has four layers:

1. **Core** (`modules/core/`): packages and settings for every host.
2. **Roles** (`modules/roles/`): one per host kind (laptop, server). A host
   imports exactly one.
3. **Features** (`modules/features/`): 22 opt-in modules (audio, btrfs, fonts,
   desktop, sway, virtualization, Steam, Firefox, containers, etc.). A role
   picks which features it needs.
4. **Hardware** (`modules/hardware/`): per-machine quirks (ThinkPad lid
   behavior, Thunderbolt dock handling, PCIe wakeup management).

Home-manager configs live in `home/dotfiles/`, one file per program: sway (274
lines), waybar (355 lines), kitty (202 lines), helix, kanshi, battery
management, fuzzel, darkman, swayidle, and more.

## Testing

Shell scripts that make runtime decisions are split into standalone files with
fixture-based test suites. There are 11 test scripts covering the lid handler,
Thunderbolt state, display layout, workspace assignment, battery level
transitions, waybar formatting, AV stepping, GPU busy detection, secrets guard,
and snapper cleanup. Every combination of docked/undocked, AC/battery, lid
open/closed has its own fixture.

A `Makefile` target runs the full suite. Every script takes overridable paths
(e.g. `LID_STATE`, `TB_DEVICES`, `DRM_DIR`) so tests substitute fixture trees
instead of reading real system state.

## ThinkPad hardware

### Lid and power

The lid handler reads four inputs: lid state, Thunderbolt authorization,
connected displays, and AC power. It outputs one of four actions: none,
stay-awake, suspend, or suspend-then-hibernate.

The decision tree: an external display or an authorized Thunderbolt device means
"docked" (stay awake). Otherwise, AC means suspend, and battery means
suspend-then-hibernate (S3 first, then S4 after 30 minutes via RTC wake).

On resume, the system re-arms PCIe wakeup, checks if the lid is still closed
(e.g. a hibernate self-wake), and if so goes straight back to sleep. Otherwise
it restarts DNS, Wi-Fi, and fires Thunderbolt recovery.

### Thunderbolt recovery

The T480s has an Alpine Ridge LP Thunderbolt controller that wedges after
hibernate. A hung ICM (internal connection manager) leaves the PCI bridge
functions alive but the NHI (native host interface) gone, or the NHI present
with an empty domain. Either way, hotplug stops working.

I wrote a detection script that checks the PCI topology for this specific
failure, and a recovery script that hard-resets the chip: it removes the PCI
functions, cuts `force_power` for a 10-second off dwell (2 seconds was not
enough), rescans, and releases. This runs automatically on resume as a systemd
service.

### Dock USB

The Lenovo USB-C dock (40AY) had SuperSpeed link failures. I added a udev rule
to hold the Alpine Ridge xHCI controller at `power/control=on` when the dock is
connected, which prevents the controller from power-gating and dropping the
link.

## Desktop (sway)

The desktop evolved from GNOME through a swaybar/i3blocks phase into the current
sway-native stack. Everything is declared in Nix.

### Waybar

I replaced swaybar/i3blocks after hitting limits with SNI tray icon rendering.
The config is written from scratch: pill-shaped blocks, Nightfox color palette,
function-matched colors (green for healthy, yellow for battery warnings). The
clock uses glibc `date` instead of the built-in widget because `libstdc++` reads
the Vancouver timezone an hour behind.

### Display and workspace management

kanshi manages output profiles with wildcard external matching. `display-plan.sh`
computes the sway output layout from connected displays, with automatic scale
detection (4K panels get 2x) and subpixel assertion. `workspace-plan.sh`
reassigns workspaces: odd numbers go to the internal panel, even to the external.
A keybind flips sides live.

### Steam Remote Play debugging

I tried streaming games from `gs-server` to the T480s via Steam Remote Play and
got a pure white screen. The streaming stats showed `display ~999 ms` (~1 FPS)
regardless of decode method.

First I thought it was the VA-API DRM hardware decoder. Toggling hardware
decoding off changed the decode time but display stayed at 999 ms. Then I tried
`LIBGL_DRI3_DISABLE=1`, which bricked Steam launch entirely because XWayland
dropped DRI2 support. Then I tried Vulkan WSI present mode overrides, since
Steam's UI uses Vulkan, not OpenGL, and all the GLX-targeted fixes were aimed at
the wrong renderer. No effect.

The actual signal was in the timing data: display time tracks inversely with how
fast frames arrive. When the server stalls and frames come in slower than 1 Hz,
display catches up and renders fine. When frames arrive at 60 Hz, display locks
to ~1000 ms. That is compositor frame-callback throttling, not a vsync timeout.
Sway sends `wl_surface.frame` callbacks for the streaming window at ~1 Hz.

### Other desktop

Custom xkb keymap remaps Caps Lock to Escape and Left Alt to Hyper_L/Mod3.
fuzzel as the launcher. darkman auto-switches GTK theme and wallpaper between
light and dark based on solar position. Wallpaper rotation renders source SVGs
to 4K PNGs at build time with an optional NixOS emblem watermark via
ImageMagick. swayidle locks after 30 minutes, then turns off displays.
dnscrypt-proxy routes DNS over HTTPS to Cloudflare.

### Battery

A systemd user timer polls battery state and fires notifications at low and
critical thresholds. The transitions are deduplicated (no repeated warnings) and
tested. At critical, the system auto-hibernates.

## Server (GPU passthrough)

The Framework server runs a Windows 11 VM via libvirt with VFIO GPU passthrough.
IOMMU is on with `iommu=pt`, and the RX 580 detaches from `amdgpu` on demand
via `virsh nodedev-detach`. The host keeps ROCm/OpenCL for compute when the GPU
is not passed through. The VM definition is declarative (NixVirt), but
`active = null` so a rebuild never disturbs a running session.

The VM sits on a libvirt NAT bridge (not macvtap, because Wi-Fi stations cannot
present a guest's MAC to the AP). A qemu hook script manages iptables port
forwarding so the guest is reachable from the LAN.

## Pi 4 media server

The Pi 4 runs Jellyfin, Sonarr, and Radarr on btrfs with subvolume quotas. The
config cross-compiles on the T480s via QEMU binfmt emulation and deploys over
the network. A test VM (`gs-pi4-vm`) boots the full Pi config in QEMU so I can
validate service changes before deploying to hardware.

The btrfs setup was its own project. Media and state are separate subvolumes
with a 256 GiB quota on media. Boot-time assertions verify the subvolumes exist
and the quota is applied. `/srv/media` automounts to survive the race where the
second device in a multi-device btrfs filesystem is not yet scanned at mount
time.

I also packaged the Moonfin Jellyfin client with its native library dependencies
(libepoxy, libXv, libmpv) and a desktop entry so it appears in fuzzel.

## OpenWrt router

The OpenWrt One runs in WISP mode: a Wi-Fi client uplink on 5 GHz through NAT
to wired LAN plus a local AP, with hardware flow offloading. The firmware image
is built declaratively from Nix using `nix-openwrt-imagebuilder`.

Tailscale joins the tailnet on first boot. LAN DNS routes through the Pi's
AdGuard Home. Wi-Fi credentials and the Tailscale auth key are encrypted with
sops-nix.

One ongoing annoyance: OpenWrt rebuilds its package indexes in place, so every
cached `packages.adb` hash goes stale on point releases. I maintain a
`freshHashes` table that patches the fixed-output derivation hashes at build
time.

## Firefox

Extensions and policies are declared in Nix. Extensions (uBlock Origin,
Bitwarden, Vimium, Privacy Badger, ClearURLs, Facebook Container, YouTube
Shorts Block, Adaptive Tab Bar Colour) are `force_installed` from AMO's latest
URL and track upstream automatically.

## Claude Code integration

The repo manages Claude Code's own configuration. I wrote custom skills as Nix
modules: a `/workflow` skill with three modes (auto, user, claude), a `/debug`
root-cause skill, and a returns-processing skill. A `sudo` workaround uses
fuzzel as an askpass agent so Claude's ttyless `sudo` calls pop a GUI prompt
instead of hanging.

## Secrets

Managed with sops-nix. Each host has an age key derived from its SSH host key.
Encrypted secrets live in `secrets/secrets.yaml` and decrypt to `/run/secrets/`
at activation.

## Repository

Private repository at `/etc/nixos`.
