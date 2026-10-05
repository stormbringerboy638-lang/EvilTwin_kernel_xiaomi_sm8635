<div align="center">
  <img src="assets/gemini-svg.svg" width="100%" alt="EvilTwin Kernel Banner" />
</div>

### **EVILTWIN KERNEL // SM8635**
*Offensive Security & Low-Level Wireless Auditing Engine for Xiaomi POCO F6 / Redmi Turbo 3 (`peridot`)*

[![Release](https://img.shields.io/github/v/release/Raju-yeager/EvilTwin_kernel_xiaomi_sm8635?color=00ff66&label=RELEASE&style=for-the-badge&logo=git&logoColor=black)](https://github.com/Raju-yeager/EvilTwin_kernel_xiaomi_sm8635/releases)
[![GKI Version](https://img.shields.io/badge/GKI-6.1.176-black?color=00ff66&style=for-the-badge&logo=linux&logoColor=00ff66)](https://kernel.org)
[![Downloads](https://img.shields.io/github/downloads/Raju-yeager/EvilTwin_kernel_xiaomi_sm8635/total?color=00e5ff&label=DOWNLOADS&style=for-the-badge&logo=cloud-download)](https://github.com/Raju-yeager/EvilTwin_kernel_xiaomi_sm8635/releases)
[![SUSFS](https://img.shields.io/badge/SUSFS-v2.3.0-purple?style=for-the-badge&logo=shield)](https://gitlab.com/simonpunk/susfs4ksu)
[![License](https://img.shields.io/badge/LICENSE-GPL--2.0-red?style=for-the-badge)](LICENSE)
[![KMI](https://img.shields.io/badge/KMI-CRC%20GATED%20IN%20CI-blue?style=for-the-badge)]()

---

> **WARNING / DISCLAIMER**  
> **USE AT YOUR OWN RISK.** This custom kernel is provided strictly for educational purposes, authorized security auditing, and hardware research. Modifying kernel-level driver states can cause bootloops, thermal variance, or system instability. The developer assumes zero liability for bricked devices, voided warranties, or misuse.

---

## ⚡ Overview & Capabilities

**EvilTwin** is an advanced GKI 6.1 kernel build customized specifically for Qualcomm SM8635 devices (`peridot`). While standard OEM kernels stub or strip raw frame hooks, EvilTwin patches Qualcomm's `qcacld-3.0` WLAN architecture to unlock complete hardware access.

* **Native Monitor Mode:** Unlocks driver transport endpoints (`wlan0`), converting the network interface into a passive listener without external USB adapters.
* **Raw Frame Transmission:** Patches `ndo_start_xmit` hooks to transmit unencrypted 802.11 frames natively.
* **Integrated Root Stack:** Features KernelSU-Next / SukiSU-Ultra with full SUSFS v2.3.0 path, mount, and symbol hiding.
* **Native Container Support:** DroidSpaces / Rootless Docker support with namespacing preserved inside reserved KABI slots.

---

## 📥 Downloads

### [⬇ Download Latest EvilTwin Kernel Release](https://github.com/Raju-yeager/EvilTwin_kernel_xiaomi_sm8635/releases/latest)

---

## 🛠 Features & Configuration

### Stack Architecture

* **Wi-Fi Subsystem:** Patched `qcacld-3.0` (`qca_cld3_qca6750`) with unlocked `con_mode` hooks and direct `mac80211` translation.
* **Root Integration:** SukiSU-Ultra / KernelSU-Next with real `KSU_VERSION` reporting.
* **Hiding Engine:** SUSFS v2.3.0 covering `sus_su`, mount points, open-redirect, and cmdline spoofing.
* **Telephony & IMS:** `DEBUG_INFO_BTF` remains enabled; full VoLTE, VoNR, and Netd functionality maintained without regressions.

---

## 🚀 Installation Guide

### Prerequisites
* Unlocked Bootloader on POCO F6 / Redmi Turbo 3 (`peridot`).
* Custom Recovery installed (AERA Recovery or OrangeFox recommended).
* Backup of current `boot`, `vendor_boot`, and `init_boot` partitions.

### Flashing via Custom Recovery
1. Download the latest `EvilTwin-peridot-release.zip` from the [Releases](https://github.com/Raju-yeager/EvilTwin_kernel_xiaomi_sm8635/releases) tab.
2. Boot into recovery (Hold **Power + Volume Up**).
3. Select **Install / Sideload**, navigate to internal storage, and pick the ZIP.
4. Swipe to flash.
5. Reboot System.

### Fastboot Method (Raw Image)
```bash
fastboot flash boot boot.img
fastboot reboot

```

---

## 🔧 Enabling Monitor Mode

Toggle monitor mode directly via root shell:

```bash
# Enable Monitor Mode
su
ip link set wlan0 down
echo 4 > /sys/module/qca_cld3_qca6750/parameters/con_mode
ip link set wlan0 up

# Return to Station (Managed) Mode
su
ip link set wlan0 down
echo 0 > /sys/module/qca_cld3_qca6750/parameters/con_mode
ip link set wlan0 up

```

---

## 🛠 Upstream Base Kernel Details

*(The following is the original documentation from the base SM8635 / peridot project by Mohithash)*

**A custom GKI kernel for the Xiaomi `peridot` — POCO F6 / Redmi Turbo 3 (Snapdragon 8s Gen 3)**

Seven root flavors. Pick the exact root + hiding stack you want!

### 01 Choose your build

| Build | Root engine | SUSFS | KPM | Manager | Best for |
| --- | --- | --- | --- | --- | --- |
| **KSUN** | KernelSU-Next v3.3.0 | — | — | KernelSU-Next | Lightweight root, no kernel-side hiding. |
| **KSUN + SUSFS** ⭐ | KernelSU-Next | v2.3.0 | — | KernelSU-Next | Root + full hiding — start here. |
| **SukiSU + SUSFS** | SukiSU-Ultra | v2.3.0 | — | KernelSU-Next | Advanced hiding + native DroidSpaces support. |
| **ReSukiSU + SUSFS** | ReSukiSU | v2.3.0 | — | KernelSU-Next | Alternative SukiSU branch with upstream patches. |
| **APatch** | APatch | — | Yes | APatch | Standard APatch kernel patching without SUSFS. |
| **APatch + SUSFS** | APatch | v2.3.0 | Yes | APatch | KPM support paired with full SUSFS path hiding. |
| **Premium** 🏆 | SukiSU-Ultra | v2.3.0 | — | KernelSU-Next | The everything-in-one image (Containers + Hiding). |

### 🏆 Premium — everything in one Image

The Premium flavor bundles the full stack on top of the base kernel:

* **Root — SukiSU-Ultra** `susfs_new` @ `278d822a` plus three kernel-side commits from `main` (pinned in `scripts/ci/pins.env`), reporting the real `KSU_VERSION` (~40800, not the `13000` fallback other builds hit).
* **Hiding — SUSFS v2.3.0:** sus paths / mounts / kstat, `uname` + cmdline spoof, open-redirect, symbol hiding.
* **Containers — DroidSpaces:** native LXC / rootless Docker via `USER_NS`, `PID_NS`, `IPC_NS` and `SYSVIPC` relocated into `ANDROID_KABI_RESERVE` slots 6/7/8 so stock `vendor_dlkm` still loads (no bootloop).
* **Calls kept working:** `DEBUG_INFO_BTF` stays enabled, so netd / IMS / VoLTE come up.

### KPM Support

SukiSU-Ultra's is stubbed upstream on GKI, so real `.kpm` support comes only from APatch/KernelPatch. APatch uses its own manager app and a superkey that is baked into the Image at build time and cannot be changed afterwards, so the APatch flavor is built only when a run supplies a private key (CI input `apatch_superkey`, or `APATCH_SUPERKEY` for local builds) and never published with a known default.

---

**Built for the Offensive Security Community.**

*Contributions, bug submissions, and patches are welcome.*
