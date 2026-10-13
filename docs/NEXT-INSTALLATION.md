# H1E firmware installation

## Status

This guide documents the exact firmware-install owner flow present in the HiDock Next development application. The normal HiDock Next application does not provide a custom-firmware picker, and this repository does not include an uploader or flashing script. Do not substitute a guessed menu, file-copy procedure, command-line uploader, or raw USB operation.

TODO: add the reviewed compatible HiDock Next branch/revision and its exact launch instructions before treating this as a public self-service procedure.

The app recognizes firmware qualification only when launched with recovery containment and the explicit firmware-qualification flag. The source flags are --recovery-mode and --firmware-qualification. A DevTools console is available only in the compatible development build; this repository does not supply the app launcher.

## Required checks

Before preparing an attempt:

1. Confirm the device is a HiDock H1E and read its current version through the application.
2. Ensure native and live recording are idle, the application device owner is available, no transfer or mass-storage session is active, and no unresolved storage-ownership journal exists.
3. Let the application perform its native card readiness check. The firmware path requires a ready card with at least 6 MiB free for these images.
4. Compare the selected release file against its exact SHA-256 and byte length in the repository manifest and SHA256SUMS.
5. Use only the current custom target, 6.3.4 / 393988. This document does not authorize routine transitions to stock firmware or downgrade to 6.3.2.

The application validates the image container, model, version, exact byte length, SHA-256, current device identity, card state, owner-idle state, and durable attempt state. Preparation is read-only. Execution is one explicit attempt and is never automatically retried.

## Development-app flow

In the compatible HiDock Next development app's DevTools console, prepare the pinned custom target:

    window.electronAPI.jensen.prepareFirmwareInstall('hardened')

Inspect the returned plan. It must be in prepared state and name target version 393988, image size 3429376, and SHA-256 4942c98f9a150f09b9a31b606e7e9d985e99b16873c5ef64e6e17667808d82e8. Confirm that the source version and device identity are the ones you intended. Do not execute a plan whose fields differ.

After that review, the application API takes the prepared attempt ID and the same exact SHA-256 once:

    window.electronAPI.jensen.executeFirmwareInstall(attemptId, '4942c98f9a150f09b9a31b606e7e9d985e99b16873c5ef64e6e17667808d82e8')

The target firmware reboots through its own update flow. Do not send a reset command, unplug the dock, or start another upload while the result is unresolved. Record the attempt ID and the application result.

After automatic reboot, observe that same attempt:

    window.electronAPI.jensen.observeFirmwareInstall(attemptId)

A confirmed result is based on the same device identity, exact reported target version, expected custom capabilities, idle state, and card readback. The dock does not attest the complete installed image hash.

## If the result is uncertain

Stop after a rejected, interrupted, or uncertain result. Preserve the durable attempt record and application evidence, and resolve the attempt through the compatible app's recovery path before making another firmware request. An upload acknowledgement alone is not proof of a completed installation.

The prior 6.3.2 image is retained for artifact history. It is not a general rollback target. The only known downgrade path in the compatible app is an explicit operator-requested return from the separately qualified 6.3.5 forward-recovery identity to 6.3.4. This repository does not provide a generic 6.3.0 restore or boot-ROM/unbrick procedure. Storage Optimize formats the recording card and is not boot-ROM recovery.
