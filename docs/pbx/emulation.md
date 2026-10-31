# Emulation and method

!!! info "Evidence tags"
    ✅ tested in emulation · 📖 read from code or tables, not exercised · 🔶 inferred · ❓ unknown · 🔌 hardware-verified (nothing yet). See the [overview](../index.md#evidence-tags).

Everything on the PBX pages was obtained by software emulation. This page says how, so that the evidence can be judged.

## Two boards, one line

An emulator of the **phone** (8051-class core, display, keypad, S0 interface controller) and an emulator of the **PBX** (ARM core, memory, S0 blocks, HDLC engine, Ethernet) run the manufacturers' own software, which the user supplies. A small *line model* connects their S0 interfaces:

| Layer | What the line does |
|---|---|
| Layer 1 | The PBX's activation command becomes INFO 2 at the phone's controller (an *activation request* indication); the phone's C/I request becomes INFO 3, which the PBX sees as its *activated* indication; the phone then gets INFO 4. Deactivation works the same way in reverse |
| D-channel | A model of the PBX's HDLC engine ([Hardware](hardware.md#the-hdlc-engine-d-channel)) takes the frames the PBX wrote into its transmit ring, and writes the phone's frames, with FCS, into its receive ring. The phone's controller exchanges frames directly |
| B-channels | The PBX's per-port sample registers exist; the phone has no voice path yet, so the line sends the idle pattern `ff` |

No part of either side's stack is replaced: the phone's layers 1 to 3 talk to the PBX's layers 1 to 3 and its call control.

## What the model leaves out

Bit-level framing, INFO timing and activation timers, FCS checking, errors and aborts, collision handling, power feeding, more than one terminal on the bus, and any B-channel audio towards the PBX.

## Observation tools

- **Frame log:** every D-channel frame in both directions with the emulated cycle count.
- **Router tap:** a breakpoint at the PBX's central message router logs each internal message (identifier, sender, receiver, port, token, length). This is how [Call processing](call-processing.md) was derived.
- **Event injection:** an internal message can be sent into the PBX through its own allocator and router, as if a feature task had sent it. The PBX then produces whatever bytes it would normally produce, which turns it into a reference encoder. The message-waiting bytes were obtained this way.
- **Regression tests:** the main observations (TEI, registration reply, an outgoing call, the message-waiting bytes) are checked automatically.

## Reproducing

You need your own phone firmware and PBX image. The generic parts (the HDLC engine, the phone's external layer 1 mode, the phone and PBX boards) are in the public emulator repository; the code that connects the two boards depends on internal data of the PBX software and is not published. The observations here are meant to be enough to repeat the experiments, or to compare a replacement against real hardware.

## Limits of the evidence

Everything is ✅ Tested in emulation only. The emulated phone and PBX run the vendors' software on models written by the author; a difference between a model and the real chip would show up only on hardware, which is why nothing here carries the 🔌 tag.
