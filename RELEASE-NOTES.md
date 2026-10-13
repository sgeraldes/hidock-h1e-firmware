# Release notes

Unofficial custom firmware for the **HiDock H1E only**. It is built on official H1E 6.3.0 and is Sebastián's own build. The version numbers 6.3.2 and 6.3.4 belong to this project and are not official HiDock releases.

## 6.3.4 — Separate microphone and system audio, controllable recording quality

Latest and recommended. Image: [firmware/H1E-6.3.4-custom.bin](firmware/H1E-6.3.4-custom.bin), 3,429,376 bytes, SHA-256 `4942c98f9a150f09b9a31b606e7e9d985e99b16873c5ef64e6e17667808d82e8`. Full notes: [releases/6.3.4.md](releases/6.3.4.md).

Compared with official 6.3.0, this firmware stops mixing your microphone and the system audio into one mono track. It keeps them as two channels and lets you choose how they are recorded.

| | Official 6.3.0 | Custom 6.3.4 |
| --- | --- | --- |
| Channels | One mono mix made before encoding | Channel 1 (left) is the microphone, Channel 2 (right) is system audio, kept apart |
| Format | Fixed | Stereo MP3 or 16-bit stereo WAV, both at 16 kHz |
| MP3 quality | Fixed | Total bitrate of 64, 96, 128 or 160 kb/s |
| Level of each source | Fixed | Recording gain per channel, 0% to 200% (0% silent, 100% unchanged) |
| Markers | Hardware button | Button, markers sent by a connected app or program, and automatic markers from sound level |
| New quality, gain and automatic-marker settings | No equivalent recording controls | Stored on the dock and used by native recording |

### What this is good for

- Record the microphone and the call audio separately and mix them later. The firmware writes one stereo file. Splitting it into two synchronized mono files is done by the HiDock Next app.
- Raise one voice without changing the other, for example the microphone at 150% and system audio at 100%.
- Keep automatic markers off by default and turn them on for the recordings that need them.

### Quality and storage

Stereo at 16 kHz, 1 MB = 1,000,000 bytes. Estimates of the audio data only, without headers, footers or markers.

| Setting | Total rate | Per hour |
| --- | --- | --- |
| MP3 profile 0 | 64 kb/s | 28.8 MB |
| MP3 profile 1 | 96 kb/s | 43.2 MB |
| MP3 profile 2 | 128 kb/s | 57.6 MB |
| MP3 profile 3 | 160 kb/s | 72 MB |
| WAV, 16-bit PCM | 64 kB/s | 230.4 MB |

The MP3 rate is the total for both channels, not per channel. WAV is uncompressed and takes four times the space of MP3 at 128 kb/s.

### Exact behavior

- Gain changes the saved recording only. It does not change USB conferencing volume or the microphone mute. Above 100% the audio can clip.
- Settings change while the dock is idle and apply to the next recording. If a stored setting cannot be read, the dock uses MP3 with automatic markers off.
- Automatic markers watch the first channel, the second channel or either one. They fire when the average absolute sound level reaches your threshold (1 to 32767), held for 100 ms. They re-arm after 100 ms of quiet and are at least 10 seconds apart. They measure loudness, not speech, and they read the audio before gain, so a channel set to 0% can still trigger a marker.
- Markers from all three sources are saved in the file footer. For app-sent markers, check the finished file and not only the dock's acknowledgement.
- Recording runs inside the dock. A recording continued with the app closed.

### Checked on a dock

- On 6.3.4, a 1 kHz tone played to the dock landed on Channel 2 about 775 times stronger than on Channel 1.
- On 6.3.4, the update rebooted by itself, all 586 recordings kept their names and sizes, and the stored settings read back unchanged.
- On 6.3.2, whose recording code 6.3.4 carries unchanged, MP3 and WAV captures decoded as expected, muting each channel worked on its own, and programmatic and automatic markers appeared in finished files.

### Limits

- The system-audio channel was confirmed with a test tone. The microphone channel follows the dock's wiring, and no separate microphone tone was played. It is not raw capture of each physical microphone in the array.
- The mapping covers normal USB recording. No claim is made for every Bluetooth mode.
- This custom update adds recording controls and markers; transcription remains an application service.
- No downgrade or ROM recovery is promised.

## 6.3.2 and 6.3.4

The audio is the same in both. 6.3.4 adds no new encoder, mixer or sound processing.

| | 6.3.2 | 6.3.4 |
| --- | --- | --- |
| Everything in the table above | Yes | Same code path |
| USB request and file-list repairs | Yes | Carried forward and rechecked |
| Console version report | 6.3.2 | 6.3.4 |
| Developer command to write a file back under its original name | No | Added, offline-tested only, not a user feature |

Choose 6.3.4 for new installations. 6.3.2 is retained as the previous released image. Its notes: [releases/6.3.2.md](releases/6.3.2.md) (3,423,232 bytes, SHA-256 `6940d83d790992d39d41791609b264d8b76041384b367076d7fbab1744c018f7`, image [firmware/H1E-6.3.2-custom.bin](firmware/H1E-6.3.2-custom.bin)).

The internal image names `restore001` (6.3.4) and `audit001` (6.3.2) identify the builds and are not feature names.

## Links

- [README](README.md)
- [PROTOCOL](PROTOCOL.md)
- [NEXT-INSTALLATION](docs/NEXT-INSTALLATION.md)
- [releases/6.3.4.md](releases/6.3.4.md)
- [releases/6.3.2.md](releases/6.3.2.md)
