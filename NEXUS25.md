# Nexus25 modem firmware

Fork of [G4KLX MMDVM](https://github.com/g4klx/MMDVM) (GPL-2.0) for
Nexus25 / CROWN P25 trunked sites. Upstream code, history and copyright
notices are kept; changes are released under the same licence. No
DVMProject code is used.

## Target boards

| Board | `make` target |
|---|---|
| Repeater Builder STM32-DVM-MTR2K / -MASTR3 (F405) | `eda405` |
| Repeater Builder STM32-DVM-MTR2K / -MASTR3 (F446) | `eda446` |
| Repeater Builder STM32-DVM / STM32_DVM_NanoPi (F446) | `dvm` |

Build: `git submodule update --init`, install `gcc-arm-none-eabi` and
`libnewlib-arm-none-eabi`, then `make <target>`; output is `bin/mmdvm_f4.bin`.

## Serial protocol additions

Frames use the normal MMDVM framing (`0xE0`, length, type, payload).
Command types `0xA0`–`0xAF` are reserved for Nexus25.

| Type | Name | Payload | Reply |
|---|---|---|---|
| `0xA0` | `NEXUS25_P25_CC_MODE` | 1 byte: bit 0 = continuous control channel on/off | ACK / NAK (3 = not duplex, 4 = bad length) |
| `0xA1` | `NEXUS25_P25_CC_IDLE` | Same layout as `MMDVM_P25_LDU`: tag byte + one complete frame (TSDU 45 bytes up to 216 bytes, with sync and NID) | ACK / NAK (4 = bad length) |
| `0xA2` | `NEXUS25_P25_CC_STATUS` | none | `E0 07 A2 <mode> <idle len> <idle count hi> <idle count lo>` |

### Continuous control channel

With CC mode on (duplex only), the modem keys up, sends the normal TX-delay
preamble once, and then never unkeys while it stays in P25 mode:

- Frames the host queues with `MMDVM_P25_HDR` / `MMDVM_P25_LDU` (TSDUs from
  the site controller's TSBK scheduler) go out first.
- When the queue is empty, the modem repeats the last `NEXUS25_P25_CC_IDLE`
  frame, so the carrier and frame sync never drop, even if the host stalls
  or loses its link to the core (site trunking).
- The idle count in `NEXUS25_P25_CC_STATUS` shows how often the host let the
  queue run dry.
- Leaving P25 mode, or sending `NEXUS25_P25_CC_MODE` with bit 0 clear,
  turns CC mode off and returns to the upstream hang-timer behaviour.

## Status

Built for the targets above; **not yet bench-tested on RF hardware**.
