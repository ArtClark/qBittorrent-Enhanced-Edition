qBittorrent Enhanced Edition
------------------------------------------
[Important Note for user and tracker operators](NOTE.md)

> **Note:** This repository is a **personal working fork** of [qBittorrent-Enhanced-Edition](https://github.com/c0re100/qBittorrent-Enhanced-Edition), created as a working copy to prepare and submit patches to qBittorrent. It is **not** and will **not** be regularly synchronized or updated with the upstream repository. For daily use, releases, and issue reporting, please use the upstream project instead.
>
> **Author:** **Art Clark** — GitHub: [ArtClark](https://github.com/ArtClark) · Email: Art.Clark@YMail.com

********************************
# Features:
1. Auto Ban Xunlei, QQ, Baidu, Xfplay, DLBT and Offline downloader

2. _Auto Ban Unknown Peer from China_ Option (Default: OFF)

3. Auto Ban BitTorrent Media Player Peer Option (Default: OFF)

4. Peer whitelist/blacklist
********************************
### Description:
qBittorrent is a bittorrent client programmed in C++ / Qt that uses
libtorrent (sometimes called libtorrent-rasterbar) by Arvid Norberg.

It aims to be a good alternative to all other bittorrent clients
out there. qBittorrent is fast, stable and provides unicode
support as well as many features.

The free [IP to Country Lite database](https://db-ip.com/db/download/ip-to-country-lite) by [DB-IP](https://db-ip.com/) is used for resolving the countries of peers. The database is licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

### Installation:

Refer to the [INSTALL](INSTALL) file.

#### Windows build toolchain for this fork

> The instructions below apply **only to this personal fork** and its Windows
> development setup. Everything is scripted: the install script records how much
> disk each component uses as it is installed (`C:\qbt-deps\sizes.log`), and the
> removal script deletes the whole toolchain again afterwards.

**Prerequisites** — everything below is **already installed on this machine**
(verified against `C:\repos\tools.json`); no action needed. The install script
takes care of the rest automatically.

* **Visual Studio 18 (Insiders)** with the C++ workload — MSVC 19.51, Windows SDK
  (10.0.26100 / 10.0.28000). Insider builds change paths weekly, so the install
  script discovers everything dynamically via the VS developer shell rather than
  hardcoding paths.
* **CMake 4.4.0** (standalone, `C:\Program Files\CMake`)
* **Ninja 1.13.2** (bundled with Visual Studio)
* **Git for Windows 2.53.0**
* **Python 3.14.6** (used by `aqtinstall` to fetch Qt)
* **Chocolatey 2.7.3** (available if ever needed)
* ~15 GB of free disk is comfortable (the retained footprint is ~4–5 GB)

What the install script adds that you do **not** already have: Qt, vcpkg and its
packages, Boost headers, and libtorrent — see the table below.

**Install** — run from **any PowerShell prompt**. The script auto-loads the
Visual Studio developer shell if `cl.exe` is not already on PATH (running it from
a VS developer prompt works too):

```pwsh
cd C:\repos\qBittorrent-Enhanced-Edition\tools
.\install-build-deps.ps1                 # Qt, vcpkg deps, Boost, libtorrent
.\install-build-deps.ps1 -BuildQbt       # same, and also builds qbt_gui (all GUI dialogs)
```

Each step appends its on-disk size and the remaining free space to
`C:\qbt-deps\sizes.log`. Expected values, measured exactly by the script:

| Component | Location | Approx. size |
| --- | --- | --- |
| Qt 6.10.1 (`win64_msvc2022_64` + qtimageformats) | `C:\Qt` | ≈ 1.3 GB |
| vcpkg (clone, bootstrap, boost-build/openssl/zlib) | `C:\vcpkg` | ≈ 0.5 GB |
| Boost 1.90.0 headers (staged via `b2`) | `C:\qbt-deps\boost` | ≈ 0.6 GB |
| libtorrent 2.0.11 | `C:\qbt-deps\libtorrent` | ≈ 0.2 GB |
| qBittorrent build | `<repo>\build` | ≈ 1 GB |

Transient build artifacts (boost archive, aqt download cache, vcpkg
buildtrees/downloads, the libtorrent intermediate `build` dir) are pruned
automatically.

**Build manually** (if you did not use `-BuildQbt`):

```pwsh
cd C:\repos\qBittorrent-Enhanced-Edition
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=RelWithDebInfo `
    -DCMAKE_PREFIX_PATH="C:\Qt\6.10.1\msvc2022_64" `
    -DCMAKE_TOOLCHAIN_FILE="C:\vcpkg\scripts\buildsystems\vcpkg.cmake" `
    -DBOOST_ROOT="C:\qbt-deps\boost\lib\cmake" `
    -DLibtorrentRasterbar_DIR="C:\qbt-deps\libtorrent\install\lib\cmake\LibtorrentRasterbar" `
    -DMSVC_RUNTIME_DYNAMIC=ON `
    -DTESTING=OFF `
    -DVCPKG_TARGET_TRIPLET=x64-windows-static-md-release
cmake --build build --target qbt_gui   # or qbt_app for the full executable
```

**Package as a PortableApps.com update ZIP** (after a successful build):

```pwsh
cd C:\repos\qBittorrent-Enhanced-Edition\tools
.\package_zip.ps1                     # -> <repo>\deploy\qBittorrentEnhancedPortable_<ver>.zip
```

The official PortableApps repack of qBittorrent-enh ships without the Qt runtimes,
so it relies on a system-installed Qt and doesn't run standalone. Our build links Qt6
dynamically, so the executable needs the Qt6 DLLs + platform/plugin DLLs in the same
folder. `package_zip.ps1` collects exactly those from the build directory and zips
them under an `App\qBittorrent64\` prefix (no enclosing per-app folder), producing
`qBittorrentEnhancedPortable_<ver>.zip` in the `deploy/` folder.

To deploy, first run the official self-installer
(`qBittorrentEnhancedPortable_<ver>.paf.exe`), let it create the per-app folder, then
extract this ZIP **into that folder** (right-drag → "Extract Here"), overwriting the
deployed app files. That updates `qbittorrent.exe` and adds the missing Qt runtime.
The launcher, AppInfo, Data, DefaultData and Other are left untouched.

**Remove everything again** (only after a successful build):

```pwsh
cd C:\repos\qBittorrent-Enhanced-Edition\tools
.\remove-build-deps.ps1            # prompts for confirmation
.\remove-build-deps.ps1 -NoPrompt  # no prompt
```

Deletes `C:\Qt`, `C:\vcpkg`, `C:\qbt-deps`, and `<repo>\build`, measuring and
reporting the freed space per component. Use `-KeepQt`, `-KeepVcpkg`, or
`-KeepQbtBuild` to retain any of them; `-RemoveAqt` additionally uninstalls
the `aqtinstall` helper via pip.

## Repository

If you are using a desktop Linux distribution without any special demands, you can use AppImage from release page.

Latest AppImage download: [qBittorrent-Enhanced-Edition-x86_64.AppImage](https://github.com/c0re100/qBittorrent-Enhanced-Edition/releases/latest/download/qBittorrent-Enhanced-Edition-x86_64.AppImage)

#### Arch Linux (Maintainer: [c0re100](https://github.com/c0re100))

[AUR](https://aur.archlinux.org/packages/qbittorrent-enhanced-git/)

[nox AUR](https://aur.archlinux.org/packages/qbittorrent-enhanced-nox-git/)

#### Debian (Maintainer: [Kolcha](https://github.com/Kolcha))

[GUI](https://software.opensuse.org//download.html?project=home%3Anikoneko%3Atest&package=qbittorrent-enhanced)

[nox](https://software.opensuse.org//download.html?project=home%3Anikoneko%3Atest&package=qbittorrent-enhanced-nox)

The one [repository](https://build.opensuse.org/project/show/home:nikoneko:test) contains all variants, links to specific packages are provided for convenience.

#### openSUSE (Maintainer: [openSUSE Chinese Community](https://github.com/openSUSE-zh))
[openSUSE repo](https://build.opensuse.org/package/show/home:opensuse_zh/qBittorrent-Enhanced-Edition)

#### openSUSE Tumbleweed - Global (Maintainer: [itachi-re](https://github.com/itachi-re))
[OBS repo](https://build.opensuse.org/package/show/home:itachi_re/qBittorrent-Enhanced-Edition)

#### Ubuntu (Maintainer: [poplite](https://github.com/poplite))

[PPA](https://launchpad.net/~poplite/+archive/ubuntu/qbittorrent-enhanced)

#### macOS (Homebrew) (Maintainer: [AlexaraWu](https://github.com/AlexaraWu))
```
brew install c0re100-qbittorrent
```

#### Windows

Windows 10 & 11 (Maintainer: [c0re100](https://github.com/c0re100))

```
winget install c0re100.qBittorrent-Enhanced-Edition
```

Chocolatey (Maintainer: [iYato](https://github.com/iYato))

```
choco install qbittorrent-enhanced
```

Scoop

```
scoop bucket add extras
scoop install qbittorrent-enhanced
```

### Misc:
For more information please visit:
https://www.qbittorrent.org

or our wiki here:
https://wiki.qbittorrent.org

Use the forum for troubleshooting before reporting bugs:
https://forum.qbittorrent.org

Please report any bug (or feature request) to:
https://bugs.qbittorrent.org

For enhanced features bug(such as Auto Ban, API, Auto Update Tracker lists...), please report to:
https://github.com/c0re100/qBittorrent-Enhanced-Edition/issues
