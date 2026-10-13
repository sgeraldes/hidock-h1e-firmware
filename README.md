# Sebastian's HiDock H1E firmware

**H1E only · Unofficial custom firmware · Latest: 6.3.4**

Keep microphone and computer playback separate in the dock's own recordings. Choose MP3 or WAV, set recording quality and each source's gain, and add native markers from the app or automatic sound-level rules.

[Detailed release notes](RELEASE-NOTES.md) · [Download releases](https://github.com/sgeraldes/hidock-h1e-firmware/releases) · [Install with HiDock Next](docs/NEXT-INSTALLATION.md)

This repository contains Sebastian Geraldes's prebuilt custom firmware images and documentation for the HiDock H1E. The firmware is based on official H1E 6.3.0. These custom version numbers are independent of HiDock's official releases. Firmware source code and app source code are kept out of this distribution repository.

## Images

| File | Firmware identity | Packed version | Size | SHA-256 |
|---|---|---:|---:|---|
| [H1E-6.3.4-custom.bin](firmware/H1E-6.3.4-custom.bin) | 6.3.4_restore001 | 393988 | 3,429,376 bytes | 4942c98f9a150f09b9a31b606e7e9d985e99b16873c5ef64e6e17667808d82e8 |
| [H1E-6.3.2-custom.bin](firmware/H1E-6.3.2-custom.bin) | 6.3.2_audit001 | 393986 | 3,423,232 bytes | 6940d83d790992d39d41791609b264d8b76041384b367076d7fbab1744c018f7 |

The 6.3.4 image is the current H1E installation; 6.3.2 is the previous custom generation. The device reports its model and version after boot, but does not report a hash of the installed flash image. The hashes above identify the repository artifacts.

## Device scope

These images are for the HiDock H1E only. They are not qualified for H1, H1 Lite, P1, P1 Mini, E1, or other devices. A matching connector or similar product name does not establish compatibility.

The custom firmware exposes native recording configuration for stereo MP3 or PCM16 stereo WAV at 16 kHz, four total MP3 bitrate choices (64, 96, 128, or 160 kb/s), independent left/right recording gains, and optional automatic native markers. The automatic marker detector uses mean PCM magnitude; it is not speech recognition or a calibrated loudness meter. See [the installation guide](docs/NEXT-INSTALLATION.md) for the supported app flow and [the protocol reference](PROTOCOL.md) for wire details.

Recording on the dock without the application, the hardware recording controls, and the basic native marker path already existed in official H1E firmware. They are not new features introduced by these images.

For USB native recording on the current custom firmware, an explicit UAC tone was observed on Channel 2 (right), about 775 times above Channel 1. Channel 1 (left) is identified as the microphone from the user's confirmed topology and the firmware's system-microphone mute behavior; a separate microphone stimulus was not run. These labels do not establish Bluetooth routing or the identity of every microphone element.

## Install and validation

Firmware installation uses the trusted HiDock Next development application's firmware owner and durable attempt journal. This repository has no uploader or flashing script. The regular HiDock Next application currently has no custom-firmware picker, so do not infer a menu or button from these documents.

Read [NEXT-INSTALLATION.md](docs/NEXT-INSTALLATION.md) before any installation attempt. It pins the public compatible Next source revision and gives the build, launch, preparation, execution and automatic-boot observation steps. Read [VALIDATION.md](docs/VALIDATION.md) for the evidence scope and its limits.

The current 6.3.4 boot was confirmed by same-device identity, version, capabilities, idle, and card readbacks after the dock rebooted automatically. This confirms an operational boot at the reported version; it is not a full installed-image hash attestation or qualification of emergency boot-ROM recovery.
