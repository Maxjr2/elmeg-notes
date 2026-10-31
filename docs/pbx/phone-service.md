# Serving a system phone

!!! info "Evidence tags"
    ✅ tested in emulation · 📖 read from code or tables, not exercised · 🔶 inferred · ❓ unknown · 🔌 hardware-verified (nothing yet). See the [overview](../index.md#evidence-tags).

What the original PBX does on the S0 line when an Elmeg system phone is connected. All entries are ✅ Tested, **PBX-observed**: the PBX's own software ran in emulation with an emulated phone attached over a simulated S0 line (see [Emulation and method](emulation.md)), in the default factory configuration. Nothing was checked on real hardware.

## Sequence from power-up

| Step | Direction | What |
|---|---|---|
| 1 | PBX | The PBX asks for activation of the line as soon as it has started (NT, INFO 2) |
| 2 | phone | The phone powers up, sees the activation request and answers with its own request (INFO 3); the line is active |
| 3 | PBX → all | An **unacknowledged broadcast** FACILITY with selector 2 and no data, global call reference (see [Codeset 7](../codeset7.md#broadcasts-pbx-observed)) |
| 4 | phone | TEI request; the PBX assigns TEI 64 |
| 5 | phone | SABME; the PBX answers UA |
| 6 | phone | The identification message (`75`), long form |
| 7 | PBX | Bare RELEASE COMPLETE, cause 81 |
| 8 | phone | Long form again, answered the same way; then the **short** form |
| 9 | PBX | The full registration reply: cause, date/time, the PBX identification, the constant `26 02 82 80` ([Registration](../registration.md)) |
| 10 | phone | STATUS, cause 95, empty codeset 7 element `02` |

After this the phone repeats the identification about every ten million emulated cycles and the PBX answers each time. It answers every keep-alive poll.

## Calls

- **Handset lifted:** the phone sends SETUP without a called number. The PBX answers with an acknowledgement on channel B1, then a broadcast FACILITY with the line state (selector 1, `04 01 82` while busy).
- **Digits:** every key is an INFORMATION message with one digit; the PBX collects them itself and decides.
- **Unassigned number:** in the default configuration the PBX clears the call with DISCONNECT, cause 3, and the phone shows *Busy, on-hook please!*. Extensions are configuration, so a real installation answers differently.
- **Hang-up:** RELEASE from the phone, RELEASE COMPLETE from the PBX, and a selector-1 broadcast with `04 01 80`.

Details of the bytes are on [Calls](../calls.md) and [Codeset 7](../codeset7.md).

## What the phone's own keys do

Pressing the phone's function keys (menus, lists, configuration) only opens screens inside the phone; **nothing reaches the PBX** apart from call set-up. The PBX-driven side of the system-phone protocol (display text, lamps, menus pushed from the PBX) is therefore only exercised when the PBX itself starts it, for example for message waiting. Which PBX events start which phone function is the main open item on the PBX side.

## Message waiting (injected test event)

Sending the PBX's message-waiting event with the text "HELLO" produced a broadcast FACILITY with selector `0a`: the text in element `02` and element `0e` with three 7-bit octets (type, kind, value). The emulated phone showed no change in its default configuration. See [Codeset 7](../codeset7.md#broadcasts-pbx-observed) for the exact bytes and how the octets varied.
