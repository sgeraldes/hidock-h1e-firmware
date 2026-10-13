# H1E command 0x000A: TTS / noise demo

Checked on Sebastian's H1E running custom 6.3.4 on 12 October 2026. This is protocol documentation, not an additional feature in the released image.

## What the command accepts

The request contains one byte. `[01]` requests demo playback; `[00]` requests cancellation. The recovered handler treats any nonzero first byte as activation. Only `01` and `00` were physically exercised. Empty input is invalid.

The command does **not** accept text, a filename, a voice selection, or an audio upload. Although its internal name is `usbiap2_set_tts_demo_ctrl`, the playback handler logs **“Play sugr noise demo”** and **“Stop sugr noise demo.”** These names do not establish speech synthesis or prove which waveform reaches the speaker.

## Internal operation

The USB handler queues an asynchronous `sugr_app` message, subtype `0x15`: event `0x8C` for activation or `0x8D` for cancellation. Its one-byte reply acknowledges command handling, not completion of playback.

Activation uses a transient RAM flag to avoid repeating an already-active request. It dispatches a registered main-task callback with value `0x51` and queues `sugr_app` event `0x4E`. The latter reaches mode-manager option 4, **music release**. The callback is indirect; its complete playback/resource selection is not yet qualified.

Cancellation clears the transient flag and, when the handler considers a demo active, queues event `0x29` to task `main`. That path cancels the current player/stream context, drains queued items, and resets its timer. It also queues `sugr_app` event `0x4F`, which reaches mode-manager option 5, **tone request**, with a default 3000 ms reservation. That reservation is not a measured three-second speaker-stop latency.

The recovered cancellation path contains no direct storage erase, firmware update, or recording-setting operation. It drains a shared audio queue; it should not be treated as a precisely targeted recording stop or a general microphone mute.

## What was observed on the dock

One activation/cancellation cycle was performed through the frozen published Next protocol implementation and a single USB owner:

| Request | Reply |
| --- | --- |
| `0x000A`, body `[01]`, sequence 16 | Same command/sequence, one-byte body `[00]` |
| `0x000A`, body `[00]`, sequence 17 | Same command/sequence, one-byte body `[00]` |

Cancellation was requested two seconds after activation. The check then allowed 3.5 seconds for the mode reservation. Automatic recording was temporarily disabled through the already-qualified settings command and restored to its original enabled value. Readback confirmed unchanged general settings, recording profile, format and marker rules; the file count remained **587**; native recording was idle and the connection remained usable. No firmware update or file-write operation was requested. File-count equality is not a byte-for-byte storage verification.

The relevant command branch, event routing, cancellation code and mode-selector branches are byte-identical between the official decoded 6.3.0 base and the frozen custom 6.3.4 image. A later, broader `sugr_app` window differs at `0x01033322–0x01033325`; that difference has not been attributed here.

**Still unqualified:** audible waveform, speech, exact acoustic stop latency, interference with concurrent notification playback, and the full dynamically registered startup callback. The ACKs and state readbacks do not prove those effects. Next has no public SDK/UI control for this command; the test used an isolated qualification helper.
