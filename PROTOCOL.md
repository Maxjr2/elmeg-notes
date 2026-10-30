# Elmeg CS410 system phone: D-channel protocol notes

Wire-format notes for the ISDN S0 (D-channel) protocol between an Elmeg CS410 system phone and an ICT-series PBX.
Purpose: interoperability, so that other equipment can serve a phone you own. The notes describe messages and frames only.
They contain no firmware, no code and no firmware addresses.

**Status labels.** CONFIRMED = reproduced against a software emulation of the phone, or read from message encoders.
INFERRED = reasoned, not verified. UNKNOWN = not decoded. Nothing here has been tested on real hardware.

## 1. Link layer (Q.921)

- S0 interface, the PBX is the network side (NT). Layer-1 activation is the normal ISDN procedure.
- SAPI 0 carries signalling. Modulo-128 operation: the phone establishes the link with SABME and only knows SABME (no SABM). CONFIRMED.
- TEI is assigned automatically. The phone sends an ID request (UI, SAPI 63, TEI 127, `0F Ri Ri 01 FF`); the network answers `0F Ri Ri 02 <TEI<<1|1>`. The phone accepts 64..126. CONFIRMED.
- Window size 1. The phone polls with `RR` commands with P=1 (about every 2 million CPU cycles of the emulated phone when idle). The network must answer each poll with an `RR` response with F=1. After four unanswered polls the phone sends SABME again and re-registers (section 4). CONFIRMED.
- A SETUP towards the phone can be sent as a broadcast UI frame (SAPI 0, TEI 127); the phone accepts it. CONFIRMED.

## 2. Layer 3 framing (Q.931)

`08 | len | call reference | message type | information elements`

- Protocol discriminator 0x08, call reference length 1. The top bit of the call reference is 0 on messages sent by the side that originated the call and 1 on messages sent by the other side. A network-originated call therefore carries flag 0 on all network messages; a phone-originated call carries flag 1 on network messages. Getting this wrong makes the phone answer RELEASE COMPLETE with cause #81 (invalid call reference). CONFIRMED.
- Message type values and the information element ids of codeset 0 are the standard DSS1 values.
- `97` is a locking shift to codeset 7 (proprietary elements). Every element after it up to the end of the message is codeset 7. CONFIRMED.
- Elements are `id | length | content`, except single-octet elements (`a1` = sending complete).

## 3. Messages the PBX can send and which elements they may carry

Order is the order on the wire. Codeset 0 elements: `04` bearer capability, `08` cause, `14` call state, `18` channel identification, `1c` facility, `1e` progress indicator, `27` notification indicator, `28` display, `29` date/time, `34` signal, `4c` connected number, `4d` connected subaddress, `6c` calling party number, `6d` calling party subaddress, `70` called party number, `71` called party subaddress, `74` redirecting number, `76` redirection number, `79` redirection subaddress, `7c` low layer compatibility, `7d` high layer compatibility, `7e` user-user. Whether an element is actually present depends on the message; this table lists what can appear (CONFIRMED from the encoder tables).

| type | name | codeset 0 | codeset 7 |
|---|---|---|---|
| 0x01 | ALERTING | 18 1c 1e 27 28 76 7e | |
| 0x02 | CALL PROCEEDING | 18 1c 1e 28 | |
| 0x03 | PROGRESS | 08 1c 1e 28 76 | |
| 0x05 | SETUP | a1 04 18 1c 1e 28 34 6c 6d 70 71 74 7c 7d 7e | 19 |
| 0x07 | CONNECT | 18 1c 1e 28 29 4c 4d 76 7c 7e | |
| 0x0D | SETUP ACKNOWLEDGE | 18 1c 1e 28 | |
| 0x0F | CONNECT ACKNOWLEDGE | 18 28 | |
| 0x21 | SUSPEND REJECT | 08 28 | |
| 0x22 | RESUME REJECT | 08 28 | |
| 0x28 | HOLD ACKNOWLEDGE | 28 | |
| 0x2D | SUSPEND ACKNOWLEDGE | 1c 28 | |
| 0x2E | RESUME ACKNOWLEDGE | 18 1c 28 | |
| 0x30 | HOLD REJECT | 08 28 | |
| 0x33 | RETRIEVE ACKNOWLEDGE | 18 28 | |
| 0x37 | RETRIEVE REJECT | 08 28 | |
| 0x45 | DISCONNECT | 08 1c 1e 28 7e | |
| 0x4D | RELEASE | 08 1c 28 7e | 11 19 25 |
| 0x5A | RELEASE COMPLETE | 08 1c 28 29 7e | 01 25 26 |
| 0x62 | FACILITY | 1c 27 28 6c 6d 70 71 76 | 02 03 04 05 08 0a 11 0b 0c 0e 22 24 25 14 1b 28 |
| 0x64 | REGISTER | 1c | 20 |
| 0x6E | NOTIFY | 27 28 76 | |
| 0x75 | STATUS ENQUIRY | 28 | |
| 0x7B | INFORMATION | a1 08 28 70 | 19 40 |
| 0x7D | STATUS | 08 14 28 | |

## 4. Registration

The phone announces itself after the data link comes up, and again after every link re-establishment, with message type 0x75 (the standard STATUS ENQUIRY value, used here for identification). CONFIRMED (emulation):

```
08 01 01 75  97  01 08 "PS130357"  0f 37 ff ff 00 00  06 56 "5.250"...  11 "12/08/15 15:04:01"  01 ff  12 ff ff ff ff ff ff ff ff ff ff ff
```

After `97`: element 01 = 8-character terminal identification; later elements carry a version string and a build date string (framing of these later elements is not fully decoded). A repeat of the message carries only element 01. The phone repeats the message about every 13 million cycles until answered.

**Reply** (PBX to phone), a RELEASE COMPLETE with the phone's call reference and the flag set:

```
08 01 81 5a | 08 02 85 d1 | 29 05 yy mm dd hh mi | 97 | 01 08 "DW130958" | 26 02 82 80
             cause #81        date/time (binary)            PBX id                fixed
```

CONFIRMED in emulation: the phone stops repeating, sets its clock from the date/time element (year, month, day, hour, minute, each one binary octet), and adds a "Tel. drctry" entry to its idle menu. Acceptance rules found by trial: the reply must be type 0x5A, must have the call reference flag set, and must contain codeset-7 element 01. A header-only reply, a reply with the flag clear, or another message type is ignored. The cause location (0x85 or 0x81) and element 26 made no visible difference. The date/time element is optional.

About 0.1 to 0.2 million cycles after an accepted reply the phone sends a STATUS: `08 01 01 7d | 08 02 80 df | 14 01 00 | 97 02 00` (cause #95, call state 0, empty codeset-7 element 02). It is sent after any accepted reply, regardless of its content, and needs no answer in the tests. CONFIRMED.

## 5. Calls (plain telephony needs only standard DSS1 plus the registration reply)

**Incoming to the phone** (all network messages carry call reference flag 0): SETUP (broadcast UI is accepted) with bearer capability `04 03 80 90 a3`, channel `18 01 81`, optional display `28`, calling number `6c`, called number `70`. The phone answers ALERTING (`01`, flag 1), and CONNECT (`07`) when the handset is lifted; the network sends CONNECT ACKNOWLEDGE. Clearing from the network: DISCONNECT with cause `08 02 80 90`; the phone answers RELEASE or RELEASE COMPLETE. CONFIRMED.

**Outgoing from the phone** (call reference flag 0 from the phone, 1 from the network): lifting the handset sends SETUP without a called number (overlap sending):

```
08 01 01 05 | 04 03 80 90 a3 | 18 01 83 | 7d 02 91 81 | 7e 01 04
```

The network answers SETUP ACKNOWLEDGE (`08 01 81 0d 18 01 89`). Every key press then arrives as INFORMATION with one digit: `08 01 01 7b 70 02 81 <ascii digit>`. On hang-up the phone sends DISCONNECT (`08 01 01 45 08 02 80 90`), the network answers RELEASE and the phone RELEASE COMPLETE. CONFIRMED.

## 6. Codeset 7 elements, network to phone

Layouts read from the PBX-side encoders. "7-bit value" means an octet with bit 7 set, `v | 0x80`. Meanings are mostly UNKNOWN: the layout is known, the purpose is not.

| id | wire layout | notes |
|---|---|---|
| 01 | `01 len data` (len up to 16) | terminal / PBX identification string. CONFIRMED purpose |
| 02 | `02 len data` | generic data blob. Purpose UNKNOWN. The phone sends it empty in its STATUS |
| 03 | `03 02 a b` | two 7-bit values |
| 04 | `04 01 a` | one 7-bit value |
| 05 | `05 02 hi lo` | 16-bit value |
| 08 | `08 01 a` | one 7-bit value |
| 0a | `0a len data` | string |
| 0b | `0b len a b c data` | three 7-bit header octets then data; whether `len` counts the header is not verified |
| 0c | `0c 01 a` | one 7-bit value |
| 0e | `0e 03 a sel b` | `a`, `b` 7-bit values; `sel` is one of 80, 81, 82, 83 |
| 11 | `11 len data` | string |
| 19 | `19 len data` | string (also appears in SETUP, RELEASE, INFORMATION) |
| 1b | `1b 01 a` | one 7-bit value |
| 20 | `20 len data` | string (used in REGISTER) |
| 22 | `22 len data` | string |
| 24 | `24 len data` | string |
| 25 | `25 01 a` | one 7-bit value |
| 26 | `26 02 82 80` | constant in the registration reply |
| 28 | `28 len a b data` | two 7-bit header octets then data |
| 40 | `40 01 a` | one 7-bit value (INFORMATION) |
| 14, 50 | not decoded | |

## 7. System facility

Messages for system-phone functions use FACILITY (0x62) with a facility element `1c 02 95 <N|0x80>`: profile octet 0x95 (system phone) and a selector N (3, 4, 7, 0x11 and 0x12 seen), followed by `97` and codeset-7 elements. The phone's REGISTER messages use the same facility element with N = 0x12 (`1c 02 95 92`). UNKNOWN: which call reference context the phone requires. FACILITY on an unknown call reference is answered with RELEASE COMPLETE cause #81 (CONFIRMED), and no codeset-7 element of the table above changed the display or LEDs when sent that way.

## 8. What is not known

Meaning of the codeset-7 elements apart from 01; what the PBX sends for key labels, lamps or directory data; the format of the phone's INFORMATION messages with codeset-7 elements on function keys; behaviour on real hardware. Corrections and captures are welcome.
