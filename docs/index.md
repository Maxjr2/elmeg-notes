# CS410 protocol notes

Wire-format notes for the ISDN D-channel protocol between an **Elmeg CS410 system phone** and an ICT-series PBX.
They exist so that other equipment (for example a small Linux box with an ISDN card) can serve a phone you own.

!!! note "Scope"
    These notes describe messages, frames and observable behaviour, in the author's own words.
    They contain no firmware, no firmware-derived code or listings, and no vendor documentation.
    The author is not affiliated with Elmeg, bintec elmeg, Funkwerk, Teldat or any successor.
    Product names are used only to say what the notes are about.

## What works with only these notes

A PBX that follows the notes can, in the author's emulation of the phone:

- bring up layer 2 (TEI assignment, SABME/UA, polling), see [Link layer](link-layer.md);
- accept the phone's registration and set its clock, see [Registration](registration.md);
- ring the phone, let it answer and clear the call, and take a call dialled from the phone digit by digit, see [Calls](calls.md).

Plain telephony needs only standard DSS1 plus the one proprietary reply described under Registration.

## What is not known

The proprietary "system phone" features (key labels, lamps, directory data) travel in a private codeset.
The byte layouts of the PBX-to-phone elements are known, their meaning mostly is not, see [Codeset 7](codeset7.md) and [Open questions](open-questions.md).

## Reading order

| Page | Content |
|---|---|
| [Link layer](link-layer.md) | Q.921: addressing, TEI, SABME, polling, what the phone drops |
| [Framing](framing.md) | Q.931 framing, call references, codesets, element formats |
| [Messages](messages.md) | Every message type, which elements it can carry, in both directions |
| [Registration](registration.md) | The identification message and the PBX reply, byte by byte |
| [Calls](calls.md) | Incoming and outgoing call flows with annotated bytes |
| [Codeset 7](codeset7.md) | The proprietary elements and the system facility |
| [Phone behaviour](phone-behaviour.md) | What the phone does on its own, menus, keys |
| [Implementing a PBX](implementing.md) | Checklist and pitfalls |
| [Open questions](open-questions.md) | What is still unknown, how to help |

## Evidence tags

Every claim in these notes carries one of these tags. **"Tested" means tested against a software emulation of the phone, not on real hardware.**

| Tag | Meaning | How much to trust it |
|---|---|---|
| ✅ **Tested** | A test PBX sent or received exactly these bytes and the emulated phone behaved as described | High for the emulated phone, not yet checked on a real one |
| 📖 **Read** | Read from the firmware's message tables or code; never run | Layouts are probably right, behaviour is untested |
| 🔶 **Inferred** | Reasoned from structure, names or analogy | Treat as a hypothesis |
| ❓ **Unknown** | Not decoded | None |
| 🔌 **Hardware** | Verified on a real phone and PBX | **No claim has this tag yet** |

All byte strings are hexadecimal.
Corrections and captures are welcome, see [Open questions](open-questions.md).

License: CC BY 4.0.
