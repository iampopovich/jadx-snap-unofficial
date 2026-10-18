# JADX Unofficial (snap)

Unofficial snap of [JADX](https://github.com/skylot/jadx), the Dex to Java
decompiler. It repackages the official release archive
(`jadx-<version>.zip` from [skylot/jadx releases](https://github.com/skylot/jadx/releases))
with OpenJDK 21 from Ubuntu 24.04, for amd64 and arm64. Not affiliated with or
endorsed by the JADX authors.

    sudo snap install jadx-unofficial
    jadx-unofficial                          # GUI
    jadx-unofficial.jadx -d out app.apk      # CLI
    sudo snap alias jadx-unofficial.jadx jadx            # optional, plain `jadx`
    sudo snap connect jadx-unofficial:removable-media    # optional, /media and /mnt

## Automatic updates

`.github/workflows/build-and-publish.yml` runs once a month (1st, 06:00 UTC);
run it manually from the Actions tab (`workflow_dispatch`) to pick up a release
sooner:

1. `check-upstream` reads the latest release tag of `skylot/jadx` and compares
   it with the version in the `stable` channel of `jadx-unofficial`.
2. If they differ, `build-and-publish` builds the snap on native amd64 and
   arm64 runners and tests it: an `ldd` sweep inside the snap runtime,
   `jadx --version`, decompiling a freshly compiled class, an HTTPS call from the
   bundled JRE (`jadx plugins --available`) and starting the GUI under Xvfb.
   Then it uploads to `edge`.
3. `promote-to-stable` promotes `edge` to `stable`.

Pushes to other branches and pull requests only build and test; publishing
happens from `main`. The version is taken from the upstream release at build
time (`adopt-info`) and the archive is checked against the sha256 digest
GitHub records for it, so new releases need no commit here.

Required repository secret: `SNAPCRAFT_STORE_CREDENTIALS`
(`snapcraft export-login --snaps=jadx-unofficial --channels=edge,stable creds.txt`).

## Limitations

- **File access.** Files are reachable in `$HOME` (non-hidden paths) and, after
  `snap connect jadx-unofficial:removable-media`, in `/media` and `/mnt`.
  Hidden folders in `$HOME` and other system paths are not visible.
- **Settings and plugins location.** Configuration and installed plugins live
  in `~/snap/jadx-unofficial/current/.config/jadx`, the cache in
  `~/snap/jadx-unofficial/common/.cache/jadx` (set through `JADX_CONFIG_DIR` /
  `JADX_CACHE_DIR`), not in `~/.config/jadx` / `~/.cache/jadx`, so they are
  separate from a non-snap install.
- **Desktop entry from the GUI.** The "create desktop entry" action of
  jadx-gui writes a launcher pointing into the snap's versioned path; use the
  menu entry the snap installs instead.
- **Update notifications.** jadx-gui may announce a new upstream version before
  the snap is rebuilt (up to a month later); update with `snap refresh`.
- **Debugger / ADB.** The snap does not ship `adb`. The debugger needs an adb
  server already running on the host (`adb start-server`), which the snap
  reaches over the network.
- **Wayland.** Java Swing runs through XWayland.
- **Unofficial.** Bugs specific to this packaging belong in this repository's
  issue tracker, not upstream.

## License

Packaging files in this repository: Apache-2.0 (see `LICENSE`).
JADX itself: Apache-2.0, Copyright 2014-2026 Skylot and contributors.
OpenJDK: GPL-2.0 with the Classpath Exception.
