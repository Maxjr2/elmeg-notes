# Messages

## PBX to phone

Order is the order on the wire. The table lists the elements a message *can* carry (CONFIRMED from the PBX's encoder tables); which ones appear depends on the message. Codeset 7 elements follow the `97` shift.

| Type | Name | Codeset 0 | Codeset 7 |
|---|---|---|---|
| `01` | ALERTING | 18 1c 1e 27 28 76 7e | |
| `02` | CALL PROCEEDING | 18 1c 1e 28 | |
| `03` | PROGRESS | 08 1c 1e 28 76 | |
| `05` | SETUP | a1 04 18 1c 1e 28 34 6c 6d 70 71 74 7c 7d 7e | 19 |
| `07` | CONNECT | 18 1c 1e 28 29 4c 4d 76 7c 7e | |
| `0d` | SETUP ACKNOWLEDGE | 18 1c 1e 28 | |
| `0f` | CONNECT ACKNOWLEDGE | 18 28 | |
| `21` | SUSPEND REJECT | 08 28 | |
| `22` | RESUME REJECT | 08 28 | |
| `28` | HOLD ACKNOWLEDGE | 28 | |
| `2d` | SUSPEND ACKNOWLEDGE | 1c 28 | |
| `2e` | RESUME ACKNOWLEDGE | 18 1c 28 | |
| `30` | HOLD REJECT | 08 28 | |
| `33` | RETRIEVE ACKNOWLEDGE | 18 28 | |
| `37` | RETRIEVE REJECT | 08 28 | |
| `45` | DISCONNECT | 08 1c 1e 28 7e | |
| `4d` | RELEASE | 08 1c 28 7e | 11 19 25 |
| `5a` | RELEASE COMPLETE | 08 1c 28 29 7e | 01 25 26 |
| `62` | FACILITY | 1c 27 28 6c 6d 70 71 76 | 02 03 04 05 08 0a 11 0b 0c 0e 22 24 25 14 1b 28 |
| `64` | REGISTER | 1c | 20 |
| `6e` | NOTIFY | 27 28 76 | |
| `75` | STATUS ENQUIRY | 28 | |
| `7b` | INFORMATION | a1 08 28 70 | 19 40 |
| `7d` | STATUS | 08 14 28 | |

## Message types the phone recognises

The phone validates incoming message types against a table: `62 05 01 02 07 0f 0d 45 4d 5a 7b 6e 7d 75 03 20 28 30 33 37 2d 21 2e 22 46 4e 60 79 64`. In names: FACILITY, SETUP, ALERTING, CALL PROCEEDING, CONNECT, CONNECT ACK, SETUP ACK, DISCONNECT, RELEASE, RELEASE COMPLETE, INFORMATION, NOTIFY, STATUS, STATUS ENQUIRY, PROGRESS, USER INFORMATION, HOLD ACK, HOLD REJECT, RETRIEVE ACK, RETRIEVE REJECT, SUSPEND ACK, SUSPEND REJECT, RESUME ACK, RESUME REJECT, RESTART, RESTART ACK, SEGMENT, CONGESTION CONTROL, REGISTER. (INFERRED, code reading of the phone.)

## Phone to PBX

The phone can send these messages (INFERRED from static reading of the phone's message builders; the ones marked CONFIRMED were seen in emulation).

| Type | Name | Notes |
|---|---|---|
| `05` | SETUP | two variants. Outgoing call: `04 03 80 90 a3`, `18 01 83`, `7d 02 91 81`, `7e 01 04`, no called number. CONFIRMED |
| `01` | ALERTING | answer to an incoming SETUP. CONFIRMED |
| `07` | CONNECT | handset lifted. CONFIRMED |
| `0f` | CONNECT ACKNOWLEDGE | |
| `45` | DISCONNECT | `08 02 80 90`. CONFIRMED |
| `4d` | RELEASE | CONFIRMED |
| `5a` | RELEASE COMPLETE | with cause `08 02 80 d1` when the call reference is unknown. CONFIRMED |
| `7b` | INFORMATION | three variants: one digit as called party number (`70 02 81 <digit>`, CONFIRMED); variants with codeset 7 elements `06`, `03`, `05` for function keys (INFERRED) |
| `7d` | STATUS | with call state. CONFIRMED |
| `75` | STATUS ENQUIRY | used for identification at registration. CONFIRMED |
| `25`, `26` | SUSPEND, RESUME | with call identity (INFERRED) |
| `24`, `31` | HOLD, RETRIEVE | (INFERRED) |
| `64` | REGISTER | five variants, each with facility `1c 02 95 92`, then `97` and codeset 7 elements `02` and `21` (INFERRED) |
| `62` | FACILITY | fourteen builders, supplementary services, content UNKNOWN |

There is no builder for CALL PROCEEDING, SETUP ACKNOWLEDGE, PROGRESS or NOTIFY, as expected for a terminal.
RESTART and RESTART ACKNOWLEDGE are accepted and answered.

## Cause values the phone uses

Standard Q.850: 16 normal clearing, 17 user busy, 30 response to STATUS ENQUIRY, 31 normal unspecified, 81 invalid call reference, 88 incompatible destination, 95 invalid message, 96 mandatory element missing, 97 message type non-existent, 99 element non-existent, 100 invalid element contents (diagnostic = element id), 102 recovery on timer expiry.
