# Layer 3 framing (Q.931)

!!! info "Evidence tags"
    ✅ tested in emulation · 📖 read from code or tables, not exercised · 🔶 inferred · ❓ unknown · 🔌 hardware-verified (nothing yet). See the [overview](index.md#evidence-tags).

```
08 | cr_len | call reference | message type | information elements ...
```

| Field | Value |
|---|---|
| Protocol discriminator | `08` (DSS1). The phone accepts `0f` for SETUP only. |
| Call reference length | `01` (one octet). A zero length means the global call reference. |
| Call reference | 7-bit number plus a flag in bit 7 |
| Message type | Standard DSS1 values, see [Messages](messages.md) |

## Call reference flag

The flag is **0 on messages sent by the side that originated the call** and **1 on messages sent by the other side**.

| Call | Network messages carry | Phone messages carry |
|---|---|---|
| Originated by the network | flag 0 (`01`) | flag 1 (`81`) |
| Originated by the phone | flag 1 (`81`) | flag 0 (`01`) |

A message with the wrong flag is treated as belonging to an unknown call. The phone answers RELEASE COMPLETE with cause #81 (`08 02 80 d1`). ✅ Tested.

## Information elements

Variable-length elements are `id | length | content`. Single-octet elements have bit 7 of the id set (`a1` = sending complete).

Codeset 0 elements are standard Q.931:

| Id | Element | Id | Element |
|---|---|---|---|
| `04` | Bearer capability | `6c` | Calling party number |
| `08` | Cause | `6d` | Calling party subaddress |
| `10` | Call identity | `70` | Called party number |
| `14` | Call state | `71` | Called party subaddress |
| `18` | Channel identification | `74` | Redirecting number |
| `1c` | Facility | `76` | Redirection number |
| `1e` | Progress indicator | `79` | Redirection subaddress |
| `27` | Notification indicator | `7c` | Low layer compatibility |
| `28` | Display | `7d` | High layer compatibility |
| `29` | Date/time | `7e` | User-user |
| `34` | Signal | `a1` | Sending complete |
| `4c` | Connected number | `4d` | Connected subaddress |

## Codeset shift

`97` is a **locking shift to codeset 7**. Every element after it, up to the end of the message, is codeset 7. ✅ Tested.
The phone also decodes codeset 5 and 6 (one element, id `1a`, content ❓ Unknown). Codeset 7 is the one that matters, see [Codeset 7](codeset7.md).

## Frequently used elements

| Element | Bytes | Meaning |
|---|---|---|
| Bearer capability | `04 03 80 90 a3` | speech, 64 kbit/s, A-law (`a2` = mu-law) |
| Channel identification | `18 01 81` / `89` / `83` | B1 preferred / B1 exclusive / any channel |
| Cause | `08 02 80 90` | location user, cause 16 (normal clearing); the phone may add a diagnostic octet |
| Cause (invalid call reference) | `08 02 80 d1` | cause 81 |
| High layer compatibility | `7d 02 91 81` | telephony |
| Called party number | `70 02 81 31` | digit "1" (octet after length: type and plan `81`, then ASCII) |
| Calling party number | `6c <len> 21 80 <digits>` | as seen in the network's own SETUP |
| Date/time | `29 05 yy mm dd hh mi` | each octet binary, not BCD |
| Display | `28 <len> text` | the phone's LCD is 21 columns wide |
