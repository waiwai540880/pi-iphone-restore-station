# pi-iphone-restore-station# Pocket iPhone Restore Station: design draft

Status: idea / design stage. No code written yet.

## Goal
A Raspberry Pi board that restores an iPhone with no laptop: plug in, press one button, watch a live progress screen.

## Components
- idevicerestore + libimobiledevice + libirecovery, built from source on 64-bit Pi OS
- Python wrapper: runs idevicerestore as a subprocess and turns its output into progress events
- Device state machine
- IPSW manager (offline cache)
- Small web UI (FastAPI): one Restore button + progress

## Device state machine
NORMAL -> DFU/RECOVERY -> RESTORING -> DONE
- Crash or USB drop during a restore -> RECOVERABLE_FAILURE: wait for the phone to reappear in DFU/recovery, then retry with backoff (max N attempts)
- Retries exhausted -> FAILED_NEEDS_USER: keep logs, show manual DFU instructions and a Retry button
- Pre-flight: refuse to start unless the IPSW passes SHA256 and is still signed

## IPSW cache (offline-first)
- Cache only the currently signed IPSW for my own iPhone model (~8 GB), on USB SSD
- When on home Ethernet/Wi-Fi: check ipsw.me for signed firmware, resumable download, SHA256 verify, then replace the old file
- Never delete the working IPSW until a restore succeeds
- UI shows "cached firmware: signed / no longer signed"

## Testing
1. pytest with fake subprocess output and simulated disconnect/reconnect sequences (no phone needed)
2. Real hardware on a spare iPhone: pull-the-cable and power-loss tests
3. Main phone last

## Known risks / open questions
- DFU/recovery USB flakiness on Pi USB controllers: unproven. Plan: powered hub, short cables, USB traces, compare Pi 4 vs Pi 5
- Apple only signs recent firmware, so cached IPSWs can expire

## Devin workflow
1. Create repo, README, this architecture doc (I review before code)
2. idevicerestore wrapper + progress parser
3. State machine + mock-USB tests
4. IPSW manager
5. Web UI
I run it on real hardware and send logs back; Devin iterates.

## Build in public
Progress and failures will be written up on my blog (WaiWai).
