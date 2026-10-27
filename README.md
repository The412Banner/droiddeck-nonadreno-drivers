# DroidDeck on Mali and Samsung Xclipse

Graphics drivers and a DroidDeck test build for phones **without** a Snapdragon (Adreno) GPU.
Experimental community testing. Not an official DroidDeck release.

- **Test build (APK):** [DroidDeck test build: non-Snapdragon](https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases/tag/droiddeck-test-non-adreno-all), built from the branch [`test/non-adreno-all`](https://github.com/Droid-Deck/DroidDeck/tree/test/non-adreno-all) of [DroidDeck](https://github.com/Droid-Deck/DroidDeck)
- **Drivers:** the [Releases page](https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases) has one release per driver, replaced in place on every rebuild
- **Progress and test results:** [PROGRESS_LOG.md](PROGRESS_LOG.md) and the test build's release notes
- **Newest test build: `948b9cbc`** (built on DroidDeck org main up to `0f839bef`, 10-27).
  - Mali bridge updated to ASEbrahim's [MaliBridge](https://github.com/ASEbrahim/MaliBridge) v0.5.4 (Dimensity 9500 texture fixes, OpenGL text fixes, fewer low-memory crashes, picture kept after a long background pause).
  - Auto uses the Mali bridge on Malis whose driver reports Vulkan 1.1 (G77, G78, G610 on older drivers); before, it turned on neither Venus nor the bridge there ("failed to find physical device").
  - gamescope p23: when the driver rejects every layout the backend offers for a flippable image, linear is tried instead of ending the session (Mali-G610 on the bridge).
  - Drivers unchanged (no new upstream releases).
  - Earlier builds added:
    - `c19b5416`: built on 0.6.0, MaliBridge v0.5.3, Auto's stand-in after a rollback / bad pair / unreadable list, the Mali display on the phone's own driver under Auto, gamescope p22, Mesa 26.2.5 Venus, PanVK-kbase 10-23 + RC4 lowmem, JM v3.0, faux123 v1.0.0, RADV-Xclipse v1.8.0
    - `c5b25920`: Auto reads the whole driver list, Venus instead of the bridge as Auto's stand-in, the bridge vkBeginCommandBuffer fix, the Venus mutex fix, Mali-G1 on Venus, low-memory notice, G615 1.1.5-FC, RADV-Xclipse v1.7.0
    - `c9ebb247`: G615 1.1.4-FC, PanVK-kbase rc3-lowmem, the 64-bit GPU address fix, screen-half libraries, driver **Update** button, bridge-to-PanVK fallback, black-screen reasons
    - `42fdbfe8`: the bridge safety net (a driver crash no longer closes the app), Venus OpenGL logging, the PowerVR overlay fix, the Xclipse label fix
    - `ab545368`: the Pixel 10 (PowerVR) regression fix, bridge picture creation, more Venus OpenGL settings, the overlay thermal fix and S24 FE crash logging
    - `48d2d52d`: the bridge memory fix, the Venus server crash fix, and OpenAL auto-install
    - `8397965d`: "GPU drawing" hidden on Mali, plus Venus screen fixes
    - `3ef49faa`: Steam OpenGL on Venus
    - `7464bd76`: the DXVK features on Venus
    - `6c7242aa`: the fixed PanVK drivers
- **⚠️ Mali phones on older builds:** don't turn on "Steam interface: GPU drawing", because it can reboot the phone. `8397965d` hides it on Mali.
- **Where things stand (devices only):**
  - **First games on Mali (Venus):** Salt and Sanctuary and Children of Morta run at ~15–30 fps on a Galaxy A15 (Mali-G57), with no glitches.
  - **Mali games on PanVK:**
    - Hollow Knight on a Lenovo Legion Tab (G615)
    - Celeste and Castle Crashers on a Motorola Edge 60 (G615), with picture glitches
    - Hades on a Xiaomi 14T Pro (G720)
  - **Samsung S24 FE (Xclipse):** games run on RADV. Monster Hunter Rise boots but crashes before its menu; R.E.P.O. and Mouse P.I. run, then an "IWaitable hung up" crash (logging added, log pending).
  - **Steam on Venus opens on every Mali tried:** Galaxy A15 and Realme Narzo 60 (G57), Motorola Edge 60 (G615), Redmi Note 13 Pro+ (G610) and Galaxy Tab S11 (G925). Steam's OpenGL runs on the GPU on the G925; on the G615/G610 it's still software (slow).
  - **Pixel 10 Pro XL (PowerVR):** Steam opens again on `ab545368`. Brotato runs at 60 fps and Slay the Spire at 40+.
  - **Mali bridge:** the crash when Steam draws (Pixel 8 Pro G715, Lenovo G615, Xiaomi G925, Tab S11) was a bridge bug, fixed in `c5b25920`.
  - **New on `c9ebb247`:** Need for Speed Most Wanted at 50–60 fps on a Redmi Note 14 Pro 5G (G615, PanVK G615 pair); HoloCure on an OPPO Find X9 (Mali-G1-Ultra, Venus); Balatro on a Pixel 9a (G715, Venus); Steam on a Pixel 8 Pro (G715, Venus).
  - **S24 FE game exits on `c9ebb247`** (Secret of the Mimic, Dreamcore, Schedule I) were Android low-memory kills, not driver faults.
  - **Galaxy A15 (G57):** Skyrim and Salt and Sanctuary play on `42fdbfe8`; desktop mode on PanVK without Venus is smooth (50–60 fps).
  - **Black screen with a JM, G615 or G710 pair?** Set Display driver to the phone's own (System) driver and send the log.
  - **From logs on `c5b25920`:** Tomb Raider (2013) at 25–30 fps on a Redmi Note 14 Pro 5G (G615, PanVK G615 1.1.5-FC); Steam Big Picture on a Pixel 8 Pro (Venus); Venus OpenGL still falls back to software on a Tab S10 Ultra (G720), as on G615/G610.
  - **From logs on `c19b5416`:** Need for Speed Underground 2 at about 40 fps on a Pixel 8 Pro (Venus); 1000xRESIST at about 14 fps on a Lenovo TB373FU (G615, bridge, low memory); the JM pair can't import Steam's screen buffers on two G77s.
  - **Waiting on retests of `948b9cbc`.**
  - Full list in the test build's release notes.
- **Get the test build only from the link above.** Other copies (for example on file-sharing sites) are not ours.

## How graphics work on each kind of phone

DroidDeck runs Steam and games inside a desktop Linux on the phone. Its normal driver, Turnip, only
works on Snapdragon. The test build adds these routes for other GPUs:

| Route | What it does | Where it comes from |
|---|---|---|
| **Open driver pairs** (PanVK for Mali, RADV for Xclipse) | An open-source Mesa driver runs Steam and games; the phone's own driver (or the pair's Android half) draws DroidDeck's screen | The drivers below, rebuilt here |
| **Mali (Venus)** | Mesa's Venus driver sends every Vulkan command to a small server in the app, which runs it on the **phone's own Mali driver**; a compatibility layer adds the features DXVK needs | [David Cimo's DroidDeck Mali port](https://github.com/DavidCimo/droiddeck-mali) (in the test build, turned on in Setup) |
| **Phone's own driver** (PowerVR, Xclipse fallback) | Steam and games use the phone's own Vulkan driver through libhybris | [DroidDeck PR #570](https://github.com/Droid-Deck/DroidDeck/pull/570) |
| **Mali bridge** | A Vulkan bridge to the phone's own Mali driver | [DroidDeck PR #517](https://github.com/Droid-Deck/DroidDeck/pull/517) |

What the test build picks automatically (every choice can be changed in the driver settings):

| Phone | Automatic choice | Other choices |
|---|---|---|
| Snapdragon (Adreno) | Turnip, unchanged | the usual Turnip versions |
| Pixel 10 / 11 (PowerVR) | the phone's own driver | none |
| Samsung Exynos (Xclipse) | RADV Xclipse (v1.6.0-RC) | RADV-Xclipse (upstream v1.8.0), the phone's own driver, Mali bridge |
| Mali v9 to v13 on Android 15+ (G57 / G68 / G77 / G78, G610 / G710, G615 / G715, G720 / G725, G925) | PanVK-kbase (zenithblue), with the phone's own driver drawing the screen | the per-chip PanVK pair, the faux123 pair on its handheld, Mali (Venus), Mali bridge |
| Mali on Android 14 or older | the per-chip PanVK pair | Mali (Venus), Mali bridge |
| Mali-G52 / G76 | FristOneRR PanVK | Mali (Venus), Mali bridge |
| Any other Mali | Mali bridge | Mali (Venus) |

Report results with **Setup > Share logs** in the DroidDeck Discord testing thread, and say which route was in use.

## The open drivers we rebuild

| Driver | GPUs, chips and devices | Source | Our release / build | Author |
|---|---|---|---|---|
| **RADV Xclipse** | Samsung Xclipse 920 / 530 / 940. Chips: Exynos 2200, Exynos 1480, Exynos 2400 / 2400e (940 is experimental). Phones: Galaxy S22 / S22+ / S22 Ultra (Exynos models), Galaxy A55, Galaxy S24 / S24+ (Exynos models), Galaxy S24 FE. Not the Snapdragon versions of these phones, and not Exynos 1580 (Xclipse 540). | [source](https://github.com/JimVulkan/radv-xclipse) · [Android releases](https://github.com/JimVulkan/radv-xclipse/releases) | [release](https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases/tag/radv-xclipse) · [build](https://github.com/The412Banner/droiddeck-nonadreno-drivers/actions/workflows/radv-xclipse.yml) | JimVulkan |
| **RADV-Xclipse (upstream v1.8.0)**, manual option | Samsung Xclipse 920 / 530 / 940. Chips: Exynos 2200, Exynos 1480, Exynos 2400 / 2400e (940 is experimental). Phones: Galaxy S22 / S22+ / S22 Ultra (Exynos models), Galaxy A55, Galaxy S24 / S24+ (Exynos models), Galaxy S24 FE. Not the Snapdragon versions of these phones, and not Exynos 1580 (Xclipse 540). | [source](https://github.com/JimVulkan/radv-xclipse) · [Android releases](https://github.com/JimVulkan/radv-xclipse/releases) | [release](https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases/tag/radv-xclipse-upstream) · [build](https://github.com/The412Banner/droiddeck-nonadreno-drivers/actions/workflows/radv-xclipse-upstream.yml) | JimVulkan |
| **PanVK-kbase (zenithblue, all newer Mali: v9 to v13)** | One driver for Mali v9 to v13 on Arm's kbase driver: G57 / G68 / G77 / G78 (v9), G610 / G710 (v10), G615 / G715 (v11), G720 (v12), G925 / G725 (v13). Author-tested (PanProbe 38/38): Lenovo Tab TB336FU (G57), Poco X6 Pro (G615), Galaxy Tab S10 Ultra (G720), Motorola Edge 40 Neo (G610), Pixel 7 / 7a (G710), Pixel 8 / 9 (G715); Galaxy Tab S11 Ultra (G925) passed on the previous beta. Chips include Helio G99, Dimensity 7300 / 8300 / 9000 / 9300 / 9400, Tensor G2 / G3 / G4. Android 15 or newer (the author's Android build needs API 35). | [source](https://github.com/zenithblue-oss/panvk-kbase-android) · [Android releases](https://github.com/zenithblue-oss/panvk-kbase-android/releases) | [release](https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases/tag/panvk-mali-kbase) · [build](https://github.com/The412Banner/droiddeck-nonadreno-drivers/actions/workflows/panvk-mali-kbase.yml) | zenithblue-oss |
| **FristOneRR PanVK (older Mali)** | Older Mali (Job Manager): G52, G57, G68, G76, G77, G78. Chips: Helio G85 / G88 / G95 / G99 / G100, Dimensity 700 / 810 / 6020 / 6080 / 6100+ / 1080 / 1200 / 1300 / 8050, Kompanio 1300T, Unisoc T820, Exynos 1280 / 1380. Phones the author lists as working: Poco M6 Pro 4G, Galaxy A15 / A15 5G / A17 4G, Realme 10 4G, Infinix Note 30 5G / Note 50 Pro, Tecno Camon 20 Pro 5G, Redmi Note 12 Pro 5G, Redmi Note 10S, Galaxy Tab S9 FE+. | [source](https://github.com/FristOneRR-Admin/FristOneRR-Panvk-Source) · [Android releases](https://github.com/FristOneRR/FristOneRR-Panvk-Driver/releases) | [release](https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases/tag/panvk-mali-jm) · [build](https://github.com/The412Banner/droiddeck-nonadreno-drivers/actions/workflows/panvk-mali-jm.yml) | FristOneRR (credits: Noysz, mexicanbr0auth, LukeValen, BossDrk) |
| **PanVK G720 (wonderkast02)** | Mali-G720 (CSF). Chips: Dimensity 9300 / 9300+, Dimensity 8400. Phones/tablets: Vivo X100 / X100 Pro, Oppo Find X7, Galaxy Tab S10+ / S10 Ultra, Poco X7 Pro, Redmi Turbo 4. | [source](https://github.com/wonderkast02/panvk-g720-kbase-csf) · [Android releases](https://github.com/wonderkast02/panvk-g720-kbase-csf/releases) | [release](https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases/tag/panvk-mali-g720) · [build](https://github.com/The412Banner/droiddeck-nonadreno-drivers/actions/workflows/panvk-mali-g720.yml) | wonderkast02 |
| **PanVK kbase Android (G615)** | Mali-G615 (CSF). Chips: Dimensity 7300 (the author's test phone is an Oppo CPH2763), Dimensity 8300. Phones: Poco X6 Pro, Redmi K70E and other Dimensity 7300 / 8300 phones. | [source](https://github.com/GunaCharanTeja/panvk-kbase-android) | [release](https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases/tag/panvk-mali-g615) · [build](https://github.com/The412Banner/droiddeck-nonadreno-drivers/actions/workflows/panvk-mali-g615.yml) | GunaCharanTeja (fork of zenithblue-oss/panvk-kbase-android) |
| **PanVK kbase Android (G610 / G710, experimental)** | Mali-G610 / G710 (CSF, arch v10). Chips: Google Tensor G2 and Dimensity 9000 / 9000+ (Mali-G710), Dimensity 7200 / 8000 / 8100 / 8200 (Mali-G610). Devices: Pixel 7 / 7 Pro / 7a, Pixel Fold, Pixel Tablet, OnePlus Nord 3, Nothing Phone (2a), Redmi Note 13 Pro+. | [source](https://github.com/GunaCharanTeja/panvk-kbase-android) | [release](https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases/tag/panvk-mali-g710) · [build](https://github.com/The412Banner/droiddeck-nonadreno-drivers/actions/workflows/panvk-mali-g710.yml) | GunaCharanTeja (fork of zenithblue-oss/panvk-kbase-android) |
| **PanVK faux123 (Retroid Pocket 4 Pro)** | Retroid Pocket 4 Pro only (Dimensity 1100, Mali-G77 MC9). Tuned and tested by the author on that handheld. | [source](https://github.com/faux123/panvk-faux123-driver-rp4pro) · [Android releases](https://github.com/faux123/panvk-faux123-driver-rp4pro/releases) | [release](https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases/tag/panvk-faux123-rp4pro) · [build](https://github.com/The412Banner/droiddeck-nonadreno-drivers/actions/workflows/faux123-mixed.yml) | faux123 (proof of concept, no support from the author) |
| **PanVK faux123 (GameForce Ace)** | GameForce Ace only (Mali-G610). Tuned and tested by the author on that handheld. | [source](https://github.com/faux123/panvk-faux123-driver-ace) · [Android releases](https://github.com/faux123/panvk-faux123-driver-ace/releases) | [release](https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases/tag/panvk-faux123-ace) · [build](https://github.com/The412Banner/droiddeck-nonadreno-drivers/actions/workflows/faux123-mixed.yml) | faux123 (proof of concept, no support from the author) |
| **PanVK faux123 (Anbernic RG556)** | Anbernic RG556 only (Unisoc T820, Mali-G57). Tuned and tested by the author on that handheld. | [source](https://github.com/faux123/panvk-faux123-driver-rg556) · [Android releases](https://github.com/faux123/panvk-faux123-driver-rg556/releases) | [release](https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases/tag/panvk-faux123-rg556) · [build](https://github.com/The412Banner/droiddeck-nonadreno-drivers/actions/workflows/faux123-mixed.yml) | faux123 (proof of concept, no support from the author) |

### Versions in the current releases (2026-10-25)

| Release | Linux half built from | Android half |
|---|---|---|
| `radv-xclipse` (default) | JimVulkan v1.6.0-RC (`05b3092`) | author's v1.6.0-RC |
| `radv-xclipse-upstream` (manual) | JimVulkan v1.8.0 (`269238e`, Mesa 27.0.0-devel, Mesa overlay pinned at `45ea3ed`): v1.6.0's fixes, BC6H at half the memory (v1.7.0), BC6H/BC7 loading ~2x faster, the BC6H signed/unsigned lighting fix, unsigned BC4 at a third of the memory | author's v1.8.0 |
| `panvk-mali-kbase` | zenithblue `9be6dd6` (10-23, csf-v11 through 222): Mali-G1-Ultra memory import fix, smaller tiler heaps and pools, X11 fixes, vkd3d fork-hang fix (main's patch 226 does not build for this profile yet) | author's RC4 lowmem (lowmem BC on by default) |
| `panvk-mali-jm` | FristOneRR source beta 1.2.0 (`efd07ba`) | author's v3.0, plus the libraries it needs beside it |
| `panvk-mali-g720` | wonderkast02 g720-development (`ca16389`) | author's 0.1.0-beta.2 |
| `panvk-mali-g615` | GunaCharanTeja 1.1.5-FC (`c52af8b`): 1.1.4-FC's Red Dead Redemption fault fixes and fewer queue stalls, plus BC textures at native size (BC1-3 to ETC2, BC4/5 to EAC) and faster BC6H/BC7 (`PANVK_BC_ETC=0` / `PANVK_BC_EAC=0` = 1.1.4 paths) | built here from the same tag, with libdrm_full beside it |
| `panvk-mali-g710` | GunaCharanTeja `aa01eb0`, profile g610-v10-csf (same profile sources as `45071e8`: newer upstream work is all in the v11 series this profile does not use) | built here from the same source, with libdrm_full beside it |
| `panvk-faux123-*` | our `panvk-mali-kbase` Linux half | faux123 v1.0.0 (one universal build, on zenithblue RC4 + his patches) |

Why two RADV builds: testers measured the final v1.6.0 about 30% slower in FauxVkTest than v1.6.0-RC, though Monster Hunter Rise reaches its menu on it. So v1.6.0-RC stays the default and the upstream build (now v1.8.0) is a manual pick. In the Linux half the author's `setprop` switches do not exist; use `RADV_XCLIPSE_BC7SRGB_RGBA8=1` and `RADV_XCLIPSE_BCSNORM_EXACT=1` instead.

Every PanVK Linux half also carries this repo's fixes: the kbase device shown as a DRM node (`patches/panvk_kbase_drm_node.py`), the per-frame file-descriptor leak fix (`patches/panvk_kbase_sync_file_fd_leak.py`; nothing to fix in the JM source) and, on CSF Mali (G610 and newer), the provoking-vertex address fix (`patches/panvk_csf_provoking_vertex_u64.py`: a 64-bit GPU address was cut to 32 bits, a GPU fault that stopped Steam's interface). The G615 / G710 Android halves built here carry it too.

Every Android half now ships the libraries its driver needs and Android does not provide (for example libdrm, zlib, zstd, libc++). Without them DroidDeck's screen silently fell back to the phone's own Mali driver. libdrm ships as `libdrm_full.so` because the phone's own `libdrm.so` would otherwise be used instead and lacks a function the driver needs. For the authors' own Android builds only the library list of the driver file is changed (libdrm.so to libdrm_full.so), never its code.

## Components

Parts the test build downloads by checksum (not for users to install by hand):

- **[Mali (Venus) component](https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases/tag/venus-proto13-e24ed8ab):**
  - Mesa's Venus Vulkan driver (Mesa 26.2.5) for DroidDeck's Linux
  - virglrenderer's render server for Android
  - the Mali compatibility layer

  Built from David Cimo's sources by the test branch's CI.
- **Patched gamescope** ([p23](https://github.com/The412Banner/droiddeck-nonadreno-drivers/releases/tag/gamescope-3.16.29-p23-nonadreno)): DroidDeck's gamescope with these additions:
  - a switch to turn off explicit sync on kernels that can't support it (Linux older than 6.6, and Mali)
  - David Cimo's patch for showing frames that arrive as plain shared-memory images (Venus)
  - running without `VK_EXT_robustness2`, which Venus doesn't offer on most Malis
  - logging which compute shader a GPU refuses, with the features it needs
  - retrying a refused screen shader without video-colour (YCbCr) samplers, and skipping them up front on Venus
  - falling back to CPU-visible memory when no GPU-local and CPU-visible type exists (Mali bridge)
  - using picture layouts the driver supports but doesn't mark as shareable (Mali bridge)
  - naming the connection that closed when it gives up with "IWaitable hung up"
  - using a plain (linear) layout when the screen backend offers none for a game's picture format, instead of crashing (Venus)
  - trying a plain (linear) layout when the driver rejects every layout the backend offers, instead of ending the session (Mali bridge on an older G610 driver)
- **Test APK signing:** `.github/workflows/test-apk-release.yml` signs a test-branch build into its four variants and replaces the release's APKs, all on GitHub.

## What this repo does

Each open driver comes as a pair, like DroidDeck's Adreno drivers:

- **Android half:** usually the author's own Android release, unchanged. G615 / G710 have no official Android build, so theirs is built here.
- **Linux half:** the same driver built here from the author's public source for DroidDeck's desktop Linux (glibc, Arch Linux ARM). It runs Steam and the games.

How the builds work:

- **Cross-compiling:** `build_linux_driver.sh` cross-compiles each source against an Arch Linux ARM sysroot.
- **Android-only forks:** where a fork removed desktop pieces (window-system loader, X11 helpers, some headers), they are restored from upstream Mesa, and small stand-ins in `compat/` replace Android's logging and property calls.
- **PanVK patches:** every PanVK build carries two patches:
  - `patches/panvk_kbase_drm_node.py`, so the Mali device shows up as a GPU to DroidDeck's screen manager
  - `patches/panvk_kbase_sync_file_fd_leak.py`, which fixes a per-frame file leak that froze the GPU after about 10 minutes
- **Workflows and releases:**
  - each driver has its own workflow (`.github/workflows/<driver>.yml`, all using `_driver.yml`) and its own release
  - `make_release.py` builds the pair and its notes from `drivers.json`
  - to rebuild one driver from a different source commit, run its workflow with `src_ref`

## Using these outside DroidDeck

The Android halves (`-Android.zip`) are standard AdrenoTools-style driver packages. They also work in Winlator-style apps such as Bannerlator, and on Mali usually go through a Vulkan wrapper. The Linux halves are only for desktop-Linux (glibc) runtimes.

## Credits

All driver and graphics work belongs to its authors. This repo only rebuilds their source for DroidDeck's Linux, repackages their Android releases as pairs, and builds the components.

**Drivers**
- **RADV Xclipse:** [JimVulkan](https://github.com/JimVulkan/radv-xclipse)
- **PanVK-kbase:** [zenithblue-oss](https://github.com/zenithblue-oss/panvk-kbase-android)
- **PanVK G615 / G710:** [GunaCharanTeja](https://github.com/GunaCharanTeja/panvk-kbase-android), a fork of zenithblue-oss's project
- **FristOneRR PanVK:** [FristOneRR](https://github.com/FristOneRR/FristOneRR-Panvk-Driver) ([source](https://github.com/FristOneRR-Admin/FristOneRR-Panvk-Source)), with work FristOneRR credits to Noysz, mexicanbr0auth, LukeValen and BossDrk
- **PanVK G720:** [wonderkast02](https://github.com/wonderkast02/panvk-g720-kbase-csf)
- **Device builds:** [faux123](https://github.com/faux123) for the Retroid Pocket 4 Pro, GameForce Ace and Anbernic RG556 (v1.0.0 is one build for all three)

**Fixes borrowed into our driver builds**
- **CSF provoking-vertex address fix** (`patches/panvk_csf_provoking_vertex_u64.py`, in every CSF PanVK build): found and proven on a Mali-G720 by [emirvapur11](https://github.com/emir34-34) in [emir34-34/DroidDeck commit 8f18a5ad](https://github.com/emir34-34/DroidDeck/commit/8f18a5ad) (repo GPL-3.0); port and write-up by [ryanistr](https://github.com/ryanistr) in [ryanistr/DroidDeck-Mali](https://github.com/ryanistr/DroidDeck-Mali/blob/main/tools/panvk-g57/patches/0004-csf-fix-provoking-vertex-fn-address-truncation.patch) (GPL-3.0). The change itself is a one-line Mesa (MIT) fix, matching upstream Mesa.
- **Android halves ship the libraries they need, with libdrm renamed libdrm_full.so** (`tools/build_android_deps.sh`, `tools/bundle_android_deps.py`): finding and on-device measurement by [ryanistr](https://github.com/ryanistr) in [ryanistr/DroidDeck-Mali](https://github.com/ryanistr/DroidDeck-Mali/blob/main/docs/development/panvk-mali-g77-bringup.md) (GPL-3.0), section 16. Shipped libraries: [libdrm](https://gitlab.freedesktop.org/mesa/drm) (MIT), [zlib](https://github.com/madler/zlib) (zlib license), [zstd](https://github.com/facebook/zstd) (BSD), libc++ from the Android NDK (Apache-2.0 with LLVM exception).

**Fixes borrowed into the DroidDeck test build**
- **Venus render server: a device's object mutex destroyed after the objects that lock it** (`tools/venus/virglrenderer-destroy-mutex-after-objects.patch`): from BattleTempest's [kevinbryant551-ux/DroidDeck-Pixel](https://github.com/kevinbryant551-ux/DroidDeck-Pixel) (GPL-3.0), `tools/venus/patches/0103-destroy-object-mutex-after-objects.patch`
- **Steam no longer marks games as background (oom_score_adj writes dropped):** idea from [jdmg28](https://github.com/jdmg28), [DroidDeck-Mali commit 6be00e00](https://github.com/jdmg28/DroidDeck-Mali/commit/6be00e00) (GPL-3.0)
- **The reason for a black screen shown on the end screen (DD_PRESENT_FAULT):** based on [jdmg28's DroidDeck-Mali](https://github.com/jdmg28/DroidDeck-Mali/blob/arena/cd8ab18a-droiddeck-mali/app/src/main/cpp/waylandcomp/src/vk_present.c) (GPL-3.0)
- **Logging which screen driver really loaded:** based on the AdrenoTools silent-fallback finding by [ryanistr](https://github.com/ryanistr), [bring-up notes section 16](https://github.com/ryanistr/DroidDeck-Mali/blob/main/docs/development/panvk-mali-g77-bringup.md) (GPL-3.0)

**Routes in the DroidDeck test build**
- **Mali (Venus):** [David Cimo](https://github.com/DavidCimo), [DavidCimo/droiddeck-mali](https://github.com/DavidCimo/droiddeck-mali)
  - the Venus port, the Mali compatibility layer, the Venus queue-submit fix, cached frame readback, and the gamescope shared-memory patch
- **PowerVR and Xclipse through the phone's own driver:** [wallentx](https://github.com/wallentx), [PR #570](https://github.com/Droid-Deck/DroidDeck/pull/570); Xclipse support in it by [stefanz92](https://github.com/stefanz92)
- **Mali bridge:** [ASEbrahim](https://github.com/ASEbrahim), [PR #517](https://github.com/Droid-Deck/DroidDeck/pull/517); the test build carries his [MaliBridge](https://github.com/ASEbrahim/MaliBridge) (GPL-3.0) as a submodule, now v0.5.4
- **Compositor safety fixes and Pixel compatibility tools:** [Leask](https://github.com/Leask), [PR #560](https://github.com/Droid-Deck/DroidDeck/pull/560), [#561](https://github.com/Droid-Deck/DroidDeck/pull/561), [#602](https://github.com/Droid-Deck/DroidDeck/pull/602)

**Foundations**
- [Mesa](https://gitlab.freedesktop.org/mesa/mesa) (MIT), including PanVK, RADV and Venus
- [virglrenderer](https://gitlab.freedesktop.org/virgl/virglrenderer)
- [gamescope](https://github.com/ValveSoftware/gamescope)
- [DroidDeck](https://github.com/Droid-Deck/DroidDeck) (GPL-3.0)
