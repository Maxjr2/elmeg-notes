# Codeset 7: proprietary elements

Everything after the `97` shift is codeset 7. The PBX uses it for system-phone functions. The layouts below are read from the PBX's message encoders (CONFIRMED). **The meaning of almost all elements is UNKNOWN.**

"7-bit value" is a single octet with bit 7 set: `v | 0x80`. `data` is raw bytes.

| Id | Layout | Notes |
|---|---|---|
| `01` | `01 len data` (len up to 16) | identification string. Phone: `PS130357`. PBX: `DW130958`. CONFIRMED |
| `02` | `02 len data` | generic data blob. The phone sends it empty in its STATUS |
| `03` | `03 02 a b` | two 7-bit values |
| `04` | `04 01 a` | one 7-bit value |
| `05` | `05 02 hi lo` | 16-bit value |
| `08` | `08 01 a` | one 7-bit value |
| `0a` | `0a len data` | string |
| `0b` | `0b len a b c data` | three 7-bit header octets, then data. Whether `len` counts the header is not verified. The phone stores up to three of these |
| `0c` | `0c 01 a` | one 7-bit value |
| `0e` | `0e 03 a sel b` | `a`, `b` 7-bit values, `sel` is `80`, `81`, `82` or `83` |
| `11` | `11 len data` | string |
| `19` | `19 len data` | string. Also appears in SETUP, RELEASE, INFORMATION |
| `1b` | `1b 01 a` | one 7-bit value |
| `20` | `20 len data` | string. Used in REGISTER |
| `22` | `22 len data` | string |
| `24` | `24 len data` | string |
| `25` | `25 01 a` | one 7-bit value |
| `26` | `26 02 82 80` | constant in the registration reply |
| `28` | `28 len a b data` | two 7-bit header octets, then data |
| `40` | `40 01 a` | one 7-bit value (INFORMATION) |
| `14`, `50` | not decoded | |

The phone's own element table has 29 ids: `01 02 03 04 05 08 0a 0b 0c 0e 11 14 19 1a 1b 20 22 24 25 26 28 30 40 ...` up to `58` (INFERRED, partly from code reading). Ids the PBX never sends (`1a`, `30`, ...) are only decoded by the phone.

## Which messages carry which elements

See the table on [Messages](messages.md). In short: **FACILITY** carries the large set `02 03 04 05 08 0a 11 0b 0c 0e 22 24 25 14 1b 28`; the others carry at most `01 11 19 20 25 26 40`.

## System facility

System-phone functions use FACILITY (`62`) with a facility element that names a function and then the codeset 7 data:

```
62  1c 02 95 <N|80>  97  <codeset 7 elements>
```

`95` is the profile octet that marks a system phone (standard supplementary services use `91`). `N` is a selector the PBX application chooses; values 3, 4, 7, `11` and `12` were seen in the PBX's code. The phone's own REGISTER messages use `N = 12` (`1c 02 95 92`).

UNKNOWN: which call context the phone requires. A FACILITY on an unknown call reference is answered with RELEASE COMPLETE cause 81, and no element in the table changed the display or LEDs when sent that way (CONFIRMED in emulation). FACILITY on a live call, or inside a call set up by the phone, has not been tried.

## Phone to PBX

INFORMATION messages from the phone carry codeset 7 elements for key events (INFERRED from the phone's message builders):

- header `97`, element `06 01 81` or `06 01 82`;
- for one sub-type also `03 02 hi lo`, with `hi`, `lo` the high and low nibble of a key code, each OR 0x80;
- for other sub-types `05 02 a b`, a 16-bit identifier, for example a key or line number.

Which function key produces which code is UNKNOWN.
