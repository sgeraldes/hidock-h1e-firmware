# HiDock H1E protocol reference

This reference covers the original 44-command Jensen namespace recovered from H1E 6.3.0 and the custom extensions present in the 6.3.2 and 6.3.4 custom images. Command numbers, behavior, and capability gates are firmware-specific. A command constant in a host application does not establish support on another model or firmware version.

## Frame format

Normal application bulk endpoints are OUT 0x01 and IN 0x82. A frame has a 12-byte header followed by its command body:

| Offset | Size | Field |
|---:|---:|---|
| 0 | 2 | Magic: 0x12 0x34 |
| 2 | 2 | Command ID, big-endian |
| 4 | 4 | Sequence/tag, big-endian |
| 8 | 4 | Body length, big-endian |
| 12 | body length | Command-specific body |

The same framing fields are big-endian. Command bodies define their own byte order. Live PCM samples are signed PCM16 little-endian; the native MP3 marker trailer uses little-endian fields. For the H1E live audio path, the host handles interleaved stereo PCM at 16 kHz. The first two body words in a live-audio response are metadata, not samples: sequence-minus-one and the system-microphone mute flag. A muted microphone flag does not mean the complete stereo body is silent.

Commands must pass through one serialized connection owner. Do not add a second USB reader, replay raw captured requests, or infer a command effect from a single response. Unless a row says otherwise, queries have an empty request body. A one-byte status reply is command-specific: an accepted or queued request does not prove persistence, completion, or a physical side effect.

## Original H1E command IDs

The table describes the original H1E 6.3.0 command namespace. Request and reply layouts are summarized at the wire boundary. For mutations, follow the exact host owner's validation and recovery policy; do not send these directly.

| ID | Command | Request body | Reply / acknowledgement | Prerequisite, limit, or unsupported behavior |
|---|---|---|---|---|
| 0x0001 | Device information | Empty | Nominally BE32 packed version plus 13-byte serial (17 bytes) | Does not return the firmware build name or installed-flash hash. Treat a short or empty reply as unknown. |
| 0x0002 | RTC read | Empty | RTC date/time representation | The client validates 7-byte BCD fields and date bounds; all-zero time remains unknown. |
| 0x0003 | RTC write | 7 BCD bytes: century, year, month, day, hour, minute, second | One-byte command status | Clock mutation; save and restore prior time and verify it through a bounded read. No implicit UTC conversion. |
| 0x0004 | File listing | Empty | 0xFFFF marker, BE32 total count, then 57-byte entries | Listing must be parsed as complete, partial, or error; do not treat a partial list as authoritative. Six bytes in each entry prefix are opaque. Entries carry file metadata including size and a first-1024-byte MD5 value. |
| 0x0005 | File transfer | Filename bytes; native path appends NUL | Raw file bytes; aligned final blocks may require a separate empty EOF frame | Use a trusted closed-file size, drain the correlated EOF, and compare downloaded bytes. The application owns cancellation and stream settlement. |
| 0x0006 | File count | Empty | BE32 count | A missing or short reply is unknown, not zero. Compare against a complete listing. |
| 0x0007 | Delete file | Filename bytes plus native NUL; H1E filename limit is 27 bytes | One-byte status: 0 success, 1 not found, other failure | Destructive. Only an explicitly owned disposable file is eligible; confirm with a refreshed catalog. Existing user recordings are excluded. |
| 0x0008 | Firmware-update request | BE32 target version, then BE32 total image bytes | One-byte status: accepted, wrong version, busy, card full, or card error | Requires native update readiness and a ready card with sufficient free space. Acceptance arms transfer; it is not an install result. |
| 0x0009 | Firmware upload | Exact selected full image, sent through the armed update flow | One-byte upload status; transfer may defer completion while staging | The OTA path reboots internally when complete. Do not add a reset or automatic retry. Upload acknowledgement is not boot confirmation. |
| 0x000A | TTS demo control | One byte: zero disables; nonzero enables | One-byte command status | Asynchronous demo events have audio/state effects. No public H1E SDK control or physical speech qualification is claimed. |
| 0x000B | Device settings read | Empty | Variable settings fields; commonly 8, 12, or 16 bytes depending on available fields | Missing optional fields remain undefined; do not fabricate defaults or imply every field is physically qualified. |
| 0x000C | Device settings write | Settings byte fields in 4-byte slots; current host setter uses a guarded 8-byte form | One-byte command status | Legacy native reads have short-body boundary risks. Use the validated application method and read settings back; status alone is not persistence proof. |
| 0x000D | File prefix transfer | BE32 requested byte count, then filename | Raw bytes from file offset zero, ending at requested count | Host path is bounded to 1 MiB and a trusted closed-file size. It does not delete or rewrite the file. |
| 0x0010 | Card information | Empty | Nominal 28 bytes: BE32 free, capacity, status, opaque value, then 12 raw card bytes | Card values are reported in MiB by the host. Check readiness before an update; do not use a card query as a recovery procedure. |
| 0x0011 | SD erase/format | Exactly 4 bytes | One-byte command status | Destructive card formatting. It erases stored recordings and is not main-firmware or boot-ROM recovery. |
| 0x0012 | Current recording filename | Empty | Current basename bytes, or an ambiguous empty reply | Empty alone does not prove idle. Corroborate with native status and the serialized owner. |
| 0x0013 | Factory settings and SD reset | Exactly 4 bytes | One-byte command status | Destructive: includes SD formatting and additional settings/UAC changes. Not a CPU reboot command. No public user workflow is provided. |
| 0x0014 | Calendar payload upload | First-fragment total length followed by length/data chunks | One-byte command status | Calendar payload schema and persistent scheduler effects are not established. Do not send arbitrary bytes. |
| 0x0015 | Partial file transfer | BE32 file offset, BE32 byte count, then filename | Raw bytes from the requested offset, ending at requested count | Host API uses trusted closed-file bounds and a 1 MiB maximum per request. This is a file-byte offset, unlike CMD26 catalog indexes. |
| 0x0016 | Tone/resource update request | 16-byte MD5, then BE32 total bytes; internal selector 4 | One-byte command status | The normal H1E native gate requires selector 9, so this selector-4 path is rejected. Recognition does not establish a usable tone updater. |
| 0x0017 | Tone/resource payload | Resource bytes for a previously armed selector-4 request | One-byte command status | No reachable normal CMD22 path is established for this selector. Peripheral update and recovery are not qualified. |
| 0x0018 | UAC update request | 16-byte MD5, then BE32 total bytes; internal selector 9 | One-byte command status | Requires exactly 1024 bytes plus native readiness and card/context checks. Peripheral recovery is separate from recording-file recovery. |
| 0x0019 | UAC payload | UAC payload bytes for an armed selector-9 request | One-byte command status | Stages the resource and requests peripheral programming/readback. Do not infer physical peripheral programming from an acknowledgement. |
| 0x001A | Cached listing page | BE32 first catalog index and BE32 requested entry count | BE32 total, then up to the requested number of 57-byte cached entries | Index is a catalog entry number, not a file offset. Current host page size is 1 to 128 and is clamped to the cached listing. |
| 0x001C | Logical key-event injection | Two bytes: logical key type and key value | One-byte command status | Only recognized key pairs are valid. The app exposes the qualified marker pair [1,3] and recording toggle [2,3]. ACK means queued, not persisted; toggle is not an idempotent stop. |
| 0x001D | Recording status and amplitude | Empty | Type:u8, name length:u8, filename, duration:BE16 seconds, level count:u8, then level bytes | Empty is ambiguous. Levels are display samples, not calibrated dB or separated channel measurements. |
| 0x0020 | Live encoding read | Empty | BE32 encoding | Not an active-state query. Rate, source identity, or channel labels require separate evidence. |
| 0x0021 | Live control | 8-byte body; action at byte 3, encoding at byte 7. Action 1 starts, 2 pauses, 0 stops. | One-byte command status | Start stops native recording and changes auto-record state before live capture. The owner must establish native idle. A pause ACK is not proof of a complete pause effect. |
| 0x0022 | Live audio transfer | Empty; deprecated offset argument is ignored | BE32 sequence-minus-one, BE32 system-microphone mute, then audio bytes | This is a stream dequeue through the sole owner. The mute field concerns the system microphone, not all audio. |
| 0xF001 | Component version information | Empty | Variable dependent version fields; full and partial replies exist | Main, DSP, UAC, IG, and Bluetooth fields can be partial. They do not attest the installed image hash. |
| 0xF002 | Key-test start | Empty | One-byte command status | Enters diagnostic behavior affecting key/microphone infrastructure. Entry and recovery are not qualified as a user operation. |
| 0xF003 | Key/potentiometer test read | Empty | Logical key byte and potentiometer byte | Depends on diagnostic state; physical mapping and calibration are not established. |
| 0xF004 | Key-test stop | Empty | One-byte command status | Stop is conditional on test state. An ACK alone does not prove all asynchronous state was restored. |
| 0xF005 | LED test | One byte, zero/nonzero | One-byte command status | Queues an LED event and may delay about 2 seconds. Physical LED mapping is not established. |
| 0xF006 | Serial read | Empty | 13 serial bytes | Identity-bearing output. Do not publish serial values in logs or reports. |
| 0xF007 | Serial write | Exactly 13 serial bytes | One-byte command status | Persistent identity mutation. No casual testing; independent NV recovery and readback are required. |
| 0xF008 | Audio-test start | One route byte; 1, 2, 3, and 5 have distinct branches; 4 is rejected | One-byte command status | Changes audio/test/recording routing. It is not a source-isolation probe or calibrated gain operation. |
| 0xF009 | Audio-test stop | Nonempty body | One-byte command status | Effects depend on test and recording state. It is not an unconditional reset. |
| 0xF00A | Diagnostic file transfer | Filename bytes | Raw diagnostic file bytes up to output capacity | Uses the SD file-read family; it is not a flash dump or arbitrary RAM read. |
| 0xF00B | Factory-state reset | Empty | One-byte command status | Clears selected state and queues events. It is not a dedicated CPU reboot opcode and not the USB lifecycle reset method. |
| 0xF00E | MFi device identifier | Empty | Four identifier bytes from a peripheral query | Involves peripheral I/O; the reply is not a general accessory or ROM protocol. |
| 0xF00F | Enter USB mass-storage application | One byte; nonzero requests entry | One-byte command status | A zero byte is not a qualified exit. Exclusive card ownership and return to Jensen are unresolved; do not use as a normal storage shortcut. |
| 0xF010 | USB write-timeout set | Intended BE32 timeout | One-byte command status | Mutates persistent host/peripheral timeout state. The native nonempty gate does not prove a complete 4-byte body. Not a Bluetooth timeout. |
| 0xF011 | USB write-timeout read | Empty | BE32 timeout | This is the USB write timeout. A deprecated Bluetooth-named host alias does not change its meaning. |

H1E does not recognize the public P1 Bluetooth/battery methods represented by host IDs 0x1001 through 0x100A. No such H1E capability should be inferred. A USB connection reset is a host lifecycle operation, not command 0xF00B or a native CPU-reboot command.

## Custom command IDs

The custom commands below are not part of official H1E firmware. Getters and setters require exact H1E custom-version qualification. The regular H1E 6.3.0 image does not expose these custom contracts. Setters require a serialized connection and idle native/live recording; they must read back values after mutation. A setter result reports an acknowledgement while persistence remains unverified until a later readback.

| ID | Contract | Request body | Reply / acknowledgement | Gate and limit |
|---|---|---|---|---|
| 0xF110 | Recording profile and gains set | 16 bytes: ASCII REA1; profile:u8 (0=64, 1=96, 2=128, 3=160 kb/s total stereo); 3 zero reserved bytes; left gain:BE32 Q15; right gain:BE32 Q15. Each gain is 0..65536 (0..200%; 32768 is unity). | One-byte status; host returns success/failed and persistence=unverified. | Exact custom H1E only. Read f111 first; idle only. Values apply to later native recordings. Amplification can clip. |
| 0xF111 | Recording profile and gains get / REA1 capability | Empty | 17 bytes: stored/fallback:u8 followed by the 16-byte REA1 record. Flag 0 means stored, 1 fallback. | A strict REA1 response is the capability gate for optional custom commands. Fallback does not prove whether a key is missing or invalid. |
| 0xF112 | Unassigned | No defined request | No defined reply | Reserved gap. Do not infer a command or capability. |
| 0xF113 | Native recording format set | 8 bytes: ASCII REF1, format:u8 (0=MP3, 1=WAV), then 3 zero reserved bytes. | One-byte status; host returns success/failed and persistence=unverified. | Requires strict REA1 and REF1 readback on the same session; idle only. The earlier REA1-only image cannot pass the REF1 gate. |
| 0xF114 | Native recording format get | Empty | 9 bytes: stored/fallback:u8 followed by the 8-byte REF1 record. | Format is MP3 or PCM16 stereo WAV at 16 kHz. Flag 0 means stored, 1 fallback; fallback defaults to MP3. |
| 0xF115 | Automatic native marker rules set | 36 bytes of little-endian u32 fields: magic 0x3633434D, version 1, size 36, enabled 0/1, lane 0/1/2, mean-magnitude threshold 1..32767, two zero reserved words, CRC32 over the first 32 bytes. | One-byte status; host returns success/failed and persistence=unverified. | Requires strict REA1 and rules readback on the same session; idle only. Lane selects first, second, or either channel. Default is disabled. |
| 0xF116 | Automatic native marker rules get | Empty | 37 bytes: stored/fallback:u8 followed by the 36-byte rules record. | Threshold is mean absolute PCM magnitude, not speech recognition or calibrated loudness. Fallback defaults to disabled. |
| 0xF117 | Read-only firmware status bridge | Exact ASCII help, version, or uptime; no NUL, newline, shell arguments, or arbitrary input. | Status byte 0 then bounded ASCII. Help lists the three names; version returns kernel=1.9.0 and firmware=6.3.2 or 6.3.4; uptime returns uptime_ms32 as a 32-bit counter. | Exact custom H1E 6.3.2/6.3.4 only, with REA1 qualification on the same connection. This is not a shell and does not attest image bytes. |
| 0xF118 | Owned scratch-file upload extension | Begin: op:u8=0, uploadId:BE32, ownerToken:BE32, totalBytes:BE32, wholeCRC32:BE32 (17 bytes). Append: op=1, uploadId, ownerToken, offset:BE32, chunkCRC32:BE32, data (17+data bytes). Commit/status/abort are op 2/4/3 with the two IDs (9 bytes). | Six bytes: ack:u8 (0 accepted, 1 rejected), state:u8, progressBytes:BE32. States 0..5 are absent/error, receiving, verify-tmp, ready, finished, verify-final. | Extension only on H1E 393986/6.3.2 and 393988/6.3.4 custom builds; 1..0x7FFFFF total bytes, chunks at most 1024 bytes, nonfinal chunks at least 512 bytes. Requires an owned session, idle audio, normal Jensen, and mass storage inactive. A finish ACK does not prove file content, power-loss atomicity, or persistence. Developer-only and parked; not an end-user feature. |
| 0xF119 | RSTR2 recording-restore extension | Capability query is op 5 (one byte). Begin op 0 is 62+filenameLength bytes: op, restoreId:BE32, ownerToken:16 bytes, totalBytes:BE32, wholeCRC32:BE32, sourceSHA256:32 bytes, filenameLength:u8, ASCII filename. Append op 1 is 29+dataLength bytes: op, restoreId, ownerToken, offset:BE32, chunkCRC32:BE32, data. Verify/commit-step op 2, abort op 3, status op 4, and recover-step op 6 are 21 bytes: op, restoreId, ownerToken. | Capability reply is 24 bytes: status, ASCII RSTR2, protocol, filename limit, BE32 file limit, BE16 chunk/nonfinal/verify/flags fields, BE32 journal limit. Operation error may be 4 bytes; bound replies carry status, state, reason:BE16, uploaded/verified/total:BE32, CRC32:BE32, expected SHA-256, filename length and filename. Host must reread the finished file and compare full SHA-256. | Exact 393988 / 6.3.4 only. Flat printable ASCII names ending in .wav or .hda only, no replacement of existing names; maximum filename 27 bytes, file 0x7FFFFFFF bytes, chunk 1024, nonfinal chunk at least 512, verification step 4096, two 512-byte journals. Requires trusted owned source, explicit restore consent, idle audio, normal Jensen and inactive mass storage. Developer-only and parked; not physically qualified as a user workflow or an end-user feature. |

The custom extensions that mutate persistent settings still require a readback. REA1/REF1/rules fallback status is a validated-record result, not a diagnosis of why storage fell back. f118 and f119 are documented for protocol completeness only; their presence does not authorize backup, restore, deletion, upload, or recovery work.

## Version boundaries

### Separate live audio on the tested H1E

Live PCM does not require a finished recording. CMD33 starts mode 2 with body `[0, 0, 0, 1, 0, 0, 0, 2]`; the existing connection owner then dequeues CMD34 with an empty body. After the eight metadata bytes, each four-byte frame contains channel 0's signed PCM16LE sample followed by channel 1's sample. The host expects 16 kHz; the protocol does not negotiate that rate.

On the author's H1E running 393988 / 6.3.4 in normal USB mode 2, a bounded live check delivered 42 packets / 67,200 frames as two independent mono buffers, matching every source byte. An explicit 1 kHz HiDock USB playback tone was 69.35 times stronger in channel 1, identifying that lane as system audio. Channel 0 is microphone in the user's fixed setup and the firmware's mute-first route. This does not qualify Bluetooth paths or individual microphone-array elements.

The check requested neither transcription nor recording-file saving. Stop was acknowledged, native recording returned idle, settings including auto-record stayed unchanged, and the file count stayed 587. Start/stop still have device-side recording effects, so consumers must establish native idle and check teardown rather than assuming that disabling host file saving disables every recording behavior.

The reusable host splitter is published in the separate [Next source commit a1256d6](https://github.com/sgeraldes/hidock-next-2/commit/a1256d653145934be53fe2dea1f09087cd09e394), with its [live channel contract](https://github.com/sgeraldes/hidock-next-2/blob/firmware/h1e-6.3.4/docs/devices/live-channel-contract.md). Consumers share the existing reader; they must not open a second CMD34 reader. Live transport already exists in official firmware; this host tap is separate from the native recording encoder changes in the custom images.

The model, exact reported version pair, active connection session, and relevant capability response form the qualification boundary. Similar version numbers on other devices are not interchangeable. In particular:

- The custom settings and firmware-status extensions are not available on official H1E 6.3.0.
- H1E marker/toggle uses CMD 0x001C key pairs; acknowledgement means the event was queued. Check the finalized native file footer to establish persisted markers.
- Live capture and native recording share device resources and require the application's sole owner.
- A missing host method is not evidence that a native command is broken; a native command in this table is not evidence that a public host API supports it.
- No protocol response in this document attests the complete installed flash image.
