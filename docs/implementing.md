# Implementing a PBX for the CS410

A checklist for building the network side. Everything here is from the notes; nothing was tested against real hardware.

## 1. Layer 1

- Act as the NT on a basic-rate S0 interface and activate the line.
- **Power:** the CS410 may take its power from the S0 bus. The current it draws is not known to the author; check the hardware or measure before choosing an interface card. The card must also support NT mode.

## 2. Layer 2

1. Answer the TEI ID request with a TEI between 64 and 126.
2. Answer SABME with UA (F = P). Reset sequence numbers.
3. Use C/R = 1 on commands and C/R = 0 on responses from the network.
4. Window size 1: wait for the acknowledgement of each I-frame.
5. Answer `RR` P = 1 with `RR` F = 1. Without this the phone drops the link after four polls.
6. Accept SABME at any time. The phone re-establishes whenever it loses patience, and then re-registers.

On Linux, an ISDN stack such as mISDN is expected to handle steps 1 to 6 and deliver layer 3 frames (not verified by the author).

## 3. Registration

1. Wait for message `75` with `97 01 08 "PS130357"`.
2. Reply with RELEASE COMPLETE: call reference of the phone with the flag set, cause, date/time, `97`, `01 08 "DW130958"`, `26 02 82 80`. See [Registration](registration.md).
3. Expect a STATUS (`7d`, cause 95, call state 0, `97 02 00`) a little later. Do not treat it as an error.
4. Repeat for every new link establishment.

## 4. Calls

- Respect the call reference flag. Your messages for a network-originated call carry flag 0, for a phone-originated call flag 1.
- Incoming: SETUP with bearer capability, channel B1, calling and called number, optional display text. Expect ALERTING, then CONNECT when the handset is lifted.
- Outgoing: be ready for SETUP **without** a called number. Answer SETUP ACKNOWLEDGE, collect the digits from INFORMATION messages and decide yourself when the number is complete.
- Clear with DISCONNECT and answer RELEASE with RELEASE COMPLETE.
- Audio is on the B channel (64 kbit/s A-law). Connect it to your own audio path (for example a SIP gateway).

## 5. Pitfalls found

| Symptom | Cause |
|---|---|
| Phone never sees your frames | wrong C/R bit |
| Phone repeats the registration message every few seconds | reply not type `5a`, flag clear, or element `01` missing |
| Link drops after a while | polls not answered with F = 1 |
| RELEASE COMPLETE cause 81 from the phone | wrong call reference flag, or the call no longer exists |
| FACILITY has no visible effect | system FACILITY needs a call context the notes do not yet describe |

## 6. Testing without hardware

The phone's behaviour can be reproduced with a software emulation of the phone, if you have your own copy of its firmware. These notes do not include any.
