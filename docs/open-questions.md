# Open questions

!!! info "Evidence tags"
    ✅ tested in emulation · 📖 read from code or tables, not exercised · 🔶 inferred · ❓ unknown · 🔌 hardware-verified (nothing yet). See the [overview](index.md#evidence-tags).

## Not known

- **A PBX-served directory.** The phone's own 250-entry book is local (see [Phone behaviour](phone-behaviour.md#directory-tel-drctry)). Whether the system-phone channel can also show entries from the PBX's central book is ❓ Unknown; no request was ever seen.
- **Settings kept in the PBX.** Which *Configuration* and *Setting* values are stored in the PBX and how the phone reads and writes them.
- **Meaning of the codeset 7 elements** (apart from `01`): key labels, lamps, directory data, ringer and display functions.
- **FACILITY context.** What call reference or state makes the phone accept a system FACILITY.
- **Phone's INFORMATION with codeset 7**: the code of each function key, and the exact content.
- **Further registration fields**: the elements after the identification string in the phone's first message.
- **Behaviour on real hardware.** Everything here was found with a software emulation and by reading message encoders; none of it has been checked on a real phone and PBX.
- **Serial module traffic.** The phone repeatedly polls a serial peer (answering-machine or expansion module) and gets no reply; what that module answers is ❓ unknown.
- **Power feed**, audio path and timing values of the real S0 bus.

## How to help

The most valuable input would be a passive capture of the D-channel between a real ICT PBX and a CS410:

- idle for a few minutes (registration and polling);
- an incoming and an outgoing call;
- every function key pressed once.

With such a capture the codeset 7 messages could be matched byte for byte against the layouts on [Codeset 7](codeset7.md). Please strip any phone numbers and identifiers before sharing a capture. Corrections to these notes are welcome through issues on the repository.

## Method

The facts come from three sources: running the phone in software emulation against a test PBX, reading the message encoder tables of the PBX, and static reading of the phone's own message handling. No firmware is included here, and none is needed to use these notes.
