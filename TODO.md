# To do (Steam Machine edition)

Tick items off as they are done; the newest entries are at the top of each list.

## Open

- [ ] **Steamify installer page, one choice key** (2026-09-28; the page works end to end, see `AGENTS.md` for the
      module/build/testing details). `packagechooserq` (custom QML) is built with each ISO by
      `build-calamares-modules.sh`. `netinstall` (checkbox tree) stays a manually swappable fallback
      (`steamify-cachyos-dev/share/sn.sh` + `nisteamify.conf`), not wired into `calamares-online.sh`.
      Still to do: a Python job (e.g. `steamifychoice`, before `packages@online`) that normalizes either page's choice
      (`packagechooser_steamifypage` GS key, or netinstall's packageOperations markers) into one `steamifyChoice`
      GS key, so `shellprocess_steamify.conf` doesn't need to know which page ran.
- [ ] **Names, the live session**: the installed system's names were checked (2026-09-30: `os-release` and `lsb-release` are
      CachyOS's, test B3). Still to look at: the installer title, Hello and About this System in the live session (only the
      installer says Steamify).
- [ ] **Test on the real Steam Machine** from a USB stick (gamescope, LEDs, CEC, power-off) before calling the ISO usable.
- [ ] **Retention, first clean-up**: the code is merged (PR #22: at most 10 dev + 10 real releases, the mirror's release with
      its ISO is deleted first). Watch the first time there are more than 10 releases of a kind; it hasn't run yet.
- [ ] **Build badge, the other states**: succeeded was seen (`v2.9.6-260930`). Failed, cancelled and a build superseded by a
      newer one haven't been seen; a crashed runner leaves the badge on "running".
- [ ] **Check with the CachyOS team** whether the installer branding and the ISO file name are fine with them.
- [ ] **Drop the Boost 1.91 workaround** in `steamify-prepare.sh` once `cachyos-calamares-next` is rebuilt against the repos' Boost.

## Done

### Release pipeline (2026-09-29 and 2026-09-30)

- [x] **Daily names like CachyOS**: tag `v<steamify>-<YYMMDD>` (dev: `v<steamify>-dev-<YYMMDD>`), file
      `steamify-cachyos-<steamify>-[dev-]<YYMMDD>-x86_64.iso`, label `STEAMIFY_<version>_<YYMMDD>`; one ISO per day and
      kind, the newest build replaces the earlier one.
- [x] **Always from `master`**: `iso-1-github-tag.yml` takes `kind` (release, dev, auto); steamify-cachyos' `bundle.yml` asks
      for `release` (a version published from main) or `dev` (an unreleased version on a release branch).
- [x] **`steamify_ref`**: a release branch builds its dev ISO with its own Steamify (and VERSION) before it is released.
- [x] **The mirror's build is started by dispatch** after the mirror has the new tag object (a replaced tag starts no run by
      itself; the dispatch needs the full ref `refs/tags/<tag>`).
- [x] **Build badge** in the GitHub release: running, then succeeded / failed / cancelled (secret `GH_RELEASE_TOKEN` on the
      mirror), linked to the public Gitea run.
- [x] **Generated release notes** and title (`CachyOS <base> with Steamify <version> (<date> UTC)`): base version, Steamify's
      changelog section, commits since the previous release; the mirror's release uses the same text.
- [x] **Retention** of at most 10 dev and 10 real releases, deleting the mirror's release with its ISO too (PR #22; first
      clean-up still to be seen, above).
- [x] **Old hand-written `CHANGELOG.md` section** (`## CachyOS with Steamify Live ISO`) removed: the release notes are generated.
- [x] **Same-day replacement of a real release** (2026-09-30): the official build for `v2.9.6-260930` replaced the test build's
      release on the mirror (ISO asset 42 -> 50, new title and generated notes, badge green with the public run link).
- [x] **Release notes** leave out the workflow and docs commits (`ci`, `docs`, `…(ci):`) in their list of ISO changes.
- [x] **The ISO is built on the Gitea mirror's runner** (`iso-2-gitea-build.yml`) and hosted there (over GitHub's 2 GB limit);
      the VM tests run locally (`scripts/vmtest.sh --install` in steamify-cachyos-dev), not on the runners.
- [x] **Build in the test VM instead of on the Steam Machine**: superseded by the runner above.

### Installer, live session and Steamify (2026-09-27 to 2026-09-29)

- [x] **Steamify PR `feat/defaults-options`**: 2.7.0 (`--options`, `--boot`) and 2.8.0 (`--defaults --list`) released; the ISO
      fetches the newest release, nothing to pass for a Steamify-side change alone.
- [x] **Full ISO install with the Steamify page** (2026-09-28, ISO VM, `--fremont`, Start in: desktop): the Summary listed
      the choices, the install step ran Steamify 2.8.0 with every item OK, `--boot desktop` applied, `exit: 0`; the
      installed system boots into the desktop.
- [x] **Steamify Summary** (2026-09-28): an unordered list of the chosen items' names plus "Starts in gaming mode/the
      desktop" (`patches/packagechooserq-steamify-summary.patch`), with `items.json` written by the launcher.
- [x] **packagechooserq built with the ISO** (2026-09-28): `build-calamares-modules.sh` builds it against the current
      `cachyos-calamares-next` (no prebuilt `.so` in git), with `built-for`; `calamares-online.sh` leaves the Steamify
      page out when the Calamares it runs differs (the step then installs everything on).
- [x] **Steamify 2.6.0** released (`--defaults`, install-time mode).
- [x] **Live session** (VM, `--fremont`): Vapor look and layout (Steam Deck wallpaper) at the live login; the power-off
      module is built for both ISO kernels and loaded at boot (in the VM it returns "No such device": no AMDI0030 GPIO
      controller, as designed; whether it keeps the real Steam Machine off is part of the hardware test).
- [x] **Regression test 2.6.0 on an existing desktop install** (test VM from ssh-ready, `--fremont`): full first run,
      re-apply, notifications/CEC/theme off and on again; no errors.
- [x] **ISO name** (first version, since superseded by the daily names above): `steamify-cachyos-<date>-x86_64.iso`.
- [x] **VM install from the ISO** (2026-09-27, `f42859a`): ran Steamify's step with every component OK, `exit: 0`, SDDM
      autologin into gamescope; the first desktop login passed too (Vapor layout, the app opened with everything on). The
      VM's text console doesn't show (virgl): read the installed disk with `qemu-nbd -r` + `mount -o ro,rescue=nologreplay,subvol=@`
      (logs in `@log`).

Build and test details: the `steam-machine-iso` skill in steamify-cachyos-dev; the release pipeline: `steamify-iso-release`.
