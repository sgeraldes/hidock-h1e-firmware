# Validation record

This record distinguishes repository artifact identity, an observed boot, and feature evidence. A device-reported version does not attest the complete bytes in flash.

## Frozen artifacts

| File | Reported identity | Bytes | SHA-256 | Status |
|---|---|---:|---|---|
| firmware/H1E-6.3.4-custom.bin | 6.3.4_restore001 / 393988 | 3,429,376 | 4942c98f9a150f09b9a31b606e7e9d985e99b16873c5ef64e6e17667808d82e8 | Current custom image; boot confirmed on the H1E |
| firmware/H1E-6.3.2-custom.bin | 6.3.2_audit001 / 393986 | 3,423,232 | 6940d83d790992d39d41791609b264d8b76041384b367076d7fbab1744c018f7 | Previous custom generation; boot confirmed on the H1E |

Both image hashes identify the exact repository artifacts. The application validates the selected file hash before sending it. The H1E does not expose an installed-flash hash, so post-boot identity and capability checks are not byte-for-byte flash attestation.

## Observed behavior and scope

| Area | Evidence | Limit |
|---|---|---|
| 6.3.4 boot | After the pinned image was sent, the device rebooted automatically. Same-device identity, reported version, capabilities, idle state, and card status were read back. | Confirms operation at reported 393988 / 6.3.4; does not attest installed image bytes or emergency recovery. |
| 6.3.4 controls | The native recording panel read MP3, 128 kb/s, both gains at 100%, and automatic rules disabled after boot. Settings were read back through the application. | Control rendering and readback do not exercise every codec, gain, source, marker, or failure combination on 6.3.4. |
| System-audio source | On 393988, an explicit HiDock UAC tone appeared on native recording Channel 2 (right), about 775 times above Channel 1. | This establishes the tested USB native system-audio route only. It does not establish every input mode. |
| Microphone source | Channel 1 (left) follows the user's confirmed H1E topology and the firmware system-microphone mute behavior. | No separate microphone stimulus was run. This is not a microphone-array or Bluetooth routing qualification. |
| Recording controls | The custom generations expose stereo MP3/WAV at 16 kHz, total MP3 rates of 64/96/128/160 kb/s, separate channel gains from 0 to 200%, and optional mean-magnitude marker rules. Physical codec, gain, and marker behavior was observed on 393986. | The earlier generation's physical feature observations are not a complete repeat qualification of 393988. Automatic rules are not speech recognition or calibrated loudness. |
| 6.3.4 programmatic marker | One app-sent native marker was read from the finalized stereo WAV at 3.716 seconds within its 10.623-second PCM duration. | Confirms this owned recording, not every concurrency or error path. |
| Clock synchronization | On 393988, one app-requested CMD3 set the clock from main-process local time. CMD2 readback was within 942 ms of the PC, with a success acknowledgement and the dock connected and idle. | This is one clock write/readback. Persistence across reboot was not tested; no firmware or recording-storage write was requested. |
| Saved-WAV marker integration | The running development app imported the current 6.3.4 capture, persisted its native marker at 3.716 s without duplication, and exported both channels with every PCM sample exact. Original and imported-file hashes stayed unchanged. | This is an existing owned file and the development app IPC path; no new recording, USB call, provider request, or full Bluetooth/source-mode qualification. |
| App-independent recording | Native dock recording can run without the app. This baseline behavior predates the custom images and was present in official firmware. | The feature is not claimed as introduced by this repository. |

The device stores custom recording configuration in nonvolatile records. Getters distinguish validated stored values from fallback values; fallback does not identify why a record could not be used. Readback confirms the value returned at that time, not every persistence or power-loss case.

## Known boundaries

- Only HiDock H1E is in scope. No compatibility is claimed for H1, H1 Lite, P1, P1 Mini, E1, or other devices.
- The custom image derives from official H1E 6.3.0. The complete vendor source tree and a reproducible full-image vendor build are not included.
- The 6.3.4 post-boot check did not attest the installed image hash.
- This repository does not qualify ROM/boot-ROM recovery or unbrick recovery.
- Storage Optimize performs card formatting; it is not a main-firmware recovery path.
- Restore, backup, and owned-file upload extensions are not documented as end-user features. Their protocol contracts and qualification gaps are recorded in [PROTOCOL.md](../PROTOCOL.md).
