# Goals and status

!!! info "Evidence tags"
    ✅ tested in emulation · 📖 read from code or tables, not exercised · 🔶 inferred · ❓ unknown · 🔌 hardware-verified (nothing yet). See the [overview](index.md#evidence-tags).

The aim of the project: **a PC with an ISDN card, running free software, serves an Elmeg system phone with all of its features**; and, as a longer-term research goal, to understand the ICT88 hardware well enough to run other software on it.

## What a replacement PBX needs

| Need | Status |
|---|---|
| Layer 1 and 2 (NT, TEI, polling) | Standard. Observed against the original PBX in emulation ✅ ([Link layer](link-layer.md)) |
| Registration and the identification reply | ✅ Tested with a test PBX and PBX-observed ([Registration](registration.md)) |
| Plain calls: incoming, outgoing, overlap dialling, clearing | ✅ Tested, PBX-observed for the outgoing case ([Calls](calls.md)) |
| Line-state broadcast, message waiting | ✅ Bytes known, effect on the phone not yet ([Codeset 7](codeset7.md)) |
| Display text, lamps, key labels, menus pushed from the PBX | ❓ Mechanism understood on the PBX side, element meanings mostly unknown |
| Key and list frames from the phone to the PBX | 📖 Layouts read, not yet exercised |
| Extensions, trunks, hold, transfer, intercom, voicemail, SMS | ❓ Not started; the PBX's call-control behaviour is the model |
| Voice (B-channel) | ❓ The phone's voice path is not modelled yet |
| Power feed for the phone | ❓ Not known; measure before choosing an interface card |

## How progress is checked

1. **Replay against the original.** The PBX's own software runs in emulation, connected to the emulated phone ([Emulation and method](pbx/emulation.md)). The same phone stimulus is later given to a replacement; the two wire transcripts must agree wherever the phone reacts.
2. **Reference encoder.** Internal events can be injected into the original PBX to obtain the exact bytes it produces; this is how the message-waiting layout was found.
3. **Hardware.** Nothing here is yet verified on a real phone and PBX. That step is only done with the author's own equipment, and a passive capture from a real installation would be the most valuable contribution (see [Open questions](open-questions.md)).

## Longer term: the PBX hardware

Understanding the hardware beyond what the emulation needs (the main chip's timers and interrupt controller, the flash layout, the update procedure, the module hardware) is a research goal, and running other software on the PBX is the long-term aim. Nothing about that is attempted on real hardware at this point, and these notes do not contain instructions for modifying equipment.
