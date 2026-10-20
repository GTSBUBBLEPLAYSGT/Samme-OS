# 👑 SammeOS

**An unofficial, fan-made, Ubuntu-based Linux distribution for Geometry Dash fans, inspired by the
energy of KingSammelot's content.** Bright neon, cubes, spikes, portals, a crown, and a clean, fast,
gaming-ready desktop. Family-friendly and positive.

> Not affiliated with or endorsed by KingSammelot, RobTop Games (Geometry Dash), Canonical/Ubuntu,
> Valve, Discord, or Mojang/Microsoft. See [Legal](#legal--copyright-notes).

---

## Features

| Area | What you get |
|------|--------------|
| Base | Ubuntu **24.04 LTS** (`noble`). One variable switches to newer LTS (e.g. 26.04 `resolute`) |
| Desktop | **GNOME** by default (Yaru-magenta-dark + neon CSS). **KDE Plasma** variant: `DESKTOP=kde` |
| Boot | Custom **GRUB theme** and **Plymouth splash** (a cube hopping over neon spikes) |
| Look | 3 original wallpapers (SVG sources included), icon theme, GTK3/GTK4 neon palette, KDE colour scheme |
| Apps in ISO | Firefox (real .deb from Mozilla), VLC, OBS Studio, GameMode, MangoHud, GParted, Timeshift, Flatpak + Software |
| Fan pack (first boot) | Steam, Discord, Prism Launcher (Minecraft), ProtonUp-Qt: installed from Flathub for *you* |
| Welcome app | Neon GTK4 app: YouTube / Twitch / Discord / Geometry Dash links, random fun messages, install buttons |
| Installer | Calamares with SammeOS branding and slideshow |
| Build | One command, plain shell (debootstrap + casper + squashfs + grub-mkrescue). No hidden magic |

## Project layout

```
SammeOS/
├── Makefile                      make deps | iso | kde | vm | clean
├── config/
│   ├── sammeos.conf              version, Ubuntu suite, desktop, paths (env-overridable)
│   ├── flatpaks.list             fan-pack apps installed at first boot
│   └── packages/                 base / gnome / kde / gaming / utils / remove .list files
├── scripts/
│   ├── build.sh                  orchestrator: assets -> rootfs -> iso
│   ├── chroot-setup.sh           runs inside the image: apt, Mozilla repo, branding, initramfs
│   ├── generate-assets.sh        SVG -> PNG for Plymouth, GRUB, wallpapers, installer
│   ├── lib.sh / test-vm.sh       helpers; boot the ISO in QEMU
├── branding/
│   ├── svg/                      ALL artwork sources (crown, cube, spike, wallpapers...)
│   ├── plymouth/                 sammeos.plymouth + sammeos.script (animation)
│   ├── grub/                     theme.txt + live-ISO grub.cfg.in
│   └── calamares/                installer branding + slideshow
├── overlay/                      copied verbatim into the image
│   ├── common/                   welcome app, first-boot script, skel (GTK css), GRUB defaults, casper.conf
│   ├── gnome/                    gschema defaults (theme, wallpaper, dock, favourites)
│   └── kde/                      colour scheme + kdeglobals
└── docs/PALETTE_AND_THEMES.md    colours, theme variants, icon/cursor notes
```

## Build the ISO

**Host:** Ubuntu/Debian (22.04+ recommended), x86-64, ~15 GB free disk, internet, root access.

```bash
git clone <your-repo> SammeOS && cd SammeOS
make deps          # installs debootstrap, squashfs-tools, xorriso, grub bits, librsvg...
make iso           # GNOME edition (≈ 20-40 min)  ->  out/SammeOS-0.1.0-gnome-amd64.iso
# or:  make kde    # KDE Plasma edition
make vm            # try it in QEMU (add --uefi via ./scripts/test-vm.sh --uefi)
```

Stages can be run individually: `sudo -E ./scripts/build.sh assets|rootfs|iso`. Re-running `rootfs`
reuses the debootstrap in `build/`; `make distclean` wipes everything for a from-scratch rebuild.

Useful overrides (all in `config/sammeos.conf`):
```bash
UBUNTU_SUITE=resolute UBUNTU_LABEL="26.04 LTS" sudo -E ./scripts/build.sh all   # newer LTS (check it exists)
SQUASHFS_COMP=xz SQUASHFS_OPTS="" sudo -E ./scripts/build.sh iso                # smaller ISO, slower
```

**Write to USB:** `sudo dd if=out/SammeOS-*.iso of=/dev/sdX bs=4M status=progress oflag=sync`
(double-check `/dev/sdX`!), or use Ventoy / balenaEtcher. **Secure Boot:** the *live* ISO uses an
unsigned GRUB, so disable Secure Boot to boot it. The *installed* system gets Ubuntu's signed shim/GRUB.

### Honest status notes
This project was written to be as straightforward as possible, but treat the first build as a
shake-down: I could only syntax-check the scripts, not run a full ISO build. Likely tweak points:
- **Calamares** (`calamares-settings-ubuntu`) is the least standardised part on Ubuntu. If the installer
  misbehaves, test with `calamares -d` in the VM and adjust `/etc/calamares/` (overlay it under `overlay/common/etc/calamares/`).
- **Wayland + root installer:** `sammeos-install` uses `xhost` + Xwayland. If it fails, log into "Ubuntu on Xorg" in the live session.
- **Brand-new suites** (26.04) may rename packages; fix the `.list` files if apt complains.

## What happens at first boot

`/etc/skel` autostarts `sammeos-firstboot` for every new user:
1. **Live session:** shows the Welcome app (with an *Install SammeOS* button).
2. **Installed system:** creates `~/Pictures/SammeOS-Wallpapers`, applies KDE look (if KDE), asks whether to
   install the **fan pack**, hints about NVIDIA drivers if needed, then opens Welcome.
It runs once per user; uncheck "Show this window at startup" in Welcome to disable. You can re-run
things manually: `sammeos-welcome`, `sammeos-install-apps --terminal`.

## Customise

| I want to... | Edit |
|--------------|------|
| Change links (YouTube/Twitch/Discord) | `overlay/common/usr/share/sammeos/links.conf` (**verify URLs and add a real Discord invite**) |
| Add/remove apps in the ISO | `config/packages/*.list` |
| Change fan-pack apps | `config/flatpaks.list` (Flathub app IDs) |
| Change fun messages | `overlay/common/usr/share/sammeos/messages.txt` |
| Change colours | `docs/PALETTE_AND_THEMES.md`, then `gtk.css`, `welcome.css`, SVGs, `theme.txt`, `SammeOS.colors` |
| Default wallpaper / dock / favourites | `overlay/gnome/.../90_sammeos.gschema.override` |
| Boot animation | `branding/plymouth/sammeos.script` (jump timing, speed, sizes are variables at the top) |
| New wallpaper | add `branding/svg/wp-NAME.svg`; it becomes `/usr/share/backgrounds/sammeos/NAME.png` automatically (add an entry to `sammeos.xml` for the GNOME picker) |
| Icon theme | add SVGs to `overlay/common/usr/share/icons/SammeOS/scalable/apps/`; inherits Yaru-magenta for everything else |
| Name/version | `config/sammeos.conf` |

Quick iteration tip: edit the SVGs, then `make assets` and preview PNGs in `build/assets/`; test Plymouth
without rebooting via `sudo plymouthd --debug; sudo plymouth --show-splash` on a spare TTY.

## Adding your own wallpapers / fan content safely

- **Personal use:** drop images in `~/Pictures/SammeOS-Wallpapers` (created at first boot) and pick them in Settings. Nothing leaves your PC.
- **Putting it in an ISO you share:** only include art that is **yours**, **CC0/public domain**, or used
  **with written permission** from the creator. "Found it on Google" is not permission.
- Keep third-party fan art out of git: add a `local-assets/` folder to `.gitignore` and keep private builds separate.
- The included wallpapers, crown, cube and spike are **original shapes** made for this project.
  The crown is a generic design, *not* KingSammelot's actual logo. If he or his team approve an official logo, swap
  `branding/svg/crown.svg` and `logo-wordmark.svg`.
- Want official branding? Ask him. Creators often love fan projects, but get it in writing.

## Legal & copyright notes

- **Unofficial fan project.** "KingSammelot", "Geometry Dash" and related names/logos belong to their owners. No such assets are included here.
- **Geometry Dash** is a paid game by RobTop Games: buy it on Steam to support the developer. SammeOS only links to it.
- **Proprietary apps are not bundled.** Steam, Discord, and Minecraft launchers are installed from Flathub on the
  *user's* machine at first boot, so you don't redistribute them. Keep it that way.
- **Ubuntu trademarks:** you may say "based on Ubuntu", but do not call your build "Ubuntu" or use Ubuntu logos.
  Review Canonical's [IP rights policy](https://ubuntu.com/legal/intellectual-property-policy) before distributing
  ISOs publicly. Some packages (e.g. certain Yaru assets, `ubuntu-wallpapers`) carry Ubuntu branding.
- **Open-source licenses:** the ISO contains GPL and other licensed software; if you distribute it, you must be able to provide source (apt source packages cover this) and keep license notices.
- Code and original art in this repo: MIT (see `LICENSE`).
- Not legal advice: when in doubt, ask.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `Missing tools: ...` | `make deps` |
| debootstrap: unknown suite | host's debootstrap is too old: install a newer one or symlink the suite script in `/usr/share/debootstrap/scripts/` |
| Leftover mounts after a crash | `sudo -E ./scripts/build.sh clean` (unmounts) |
| ISO > 4 GiB squashfs warning | trim `gaming.list`/`utils.list`, or `SQUASHFS_COMP=xz` |
| Black screen in VM | use the *safe graphics* entry, enable 3D acceleration (virtio) |
| No Plymouth splash | check `plymouth-label` installed, `update-alternatives --display default.plymouth`, rerun `update-initramfs -u` |
| GRUB theme missing on the *live* ISO | confirm `/boot/grub/themes/sammeos/` exists in the ISO (`xorriso -indev out/*.iso -ls /boot/grub/themes`) |

Have fun, be kind, and good luck on your next run! 🟪🟦🟩
