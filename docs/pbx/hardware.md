# Hardware

!!! info "Evidence tags"
    ✅ tested in emulation · 📖 read from code or tables, not exercised · 🔶 inferred · ❓ unknown · 🔌 hardware-verified (nothing yet). See the [overview](../index.md#evidence-tags).

What the PBX board provides, as far as the emulation needs it. Register-level facts below were found by reading how the PBX software uses the hardware and by running it; there is no datasheet for the main chip, and nothing was checked on a real board (🔌 none).

## Overview

| Part | What is known |
|---|---|
| CPU | ARM with Thumb interworking, little endian, ARM7TDMI-class behaviour. Interrupt and fast-interrupt lines are used; the fast interrupt is the frame tick that moves B-channel samples |
| Memory | RAM of about 12 MiB; NOR flash holding the program and the configuration. The program is stored compressed and unpacked at start-up by its own loader |
| Line interfaces | Two integrated S/T interfaces on the main chip, plus up to three more ports on serial-attached ISDN transceiver chips |
| Ethernet | One MAC with descriptor DMA and an MDIO-managed PHY. A private framing carries management messages (see below) |
| USB | A device controller; one endpoint is a serial console |
| Optional modules | Router, analog, DECT; the PBX detects them at start-up |

## The integrated S0 interfaces

Two blocks of 0x400 bytes each, at 0x170000 and 0x170400. ✅ Tested, PBX-observed (the PBX's own driver ran against a model with exactly these registers):

| Offset | Meaning |
|---|---|
| 0x00 / 0x08 | B1 / B2 receive sample (read) |
| 0x04 / 0x0c | B1 / B2 transmit sample (write) |
| 0xc0 | C/I: a read gives the line **indication** in the low nibble, a write sets the **command**. They are different registers behind one address |
| 0xc4, 0xc8, 0xdc | mode and clock set-up written by the driver (TE and NT values differ) |

Commands the driver writes to 0xc0: `60` activate as terminal (TE), `e0` activate as network (NT), `40` deactivate. Indications that matter in NT mode: `3` = activated (INFO 3 seen), `1` = deactivated. A TE reads `7` for an activation requested by the network and has further states for INFO 4.

Which block is NT and which TE is a configuration choice: in the default set-up the first is NT (towards phones) and the second TE (towards a network).

!!! note "B-channel switching"
    There is no switch matrix in hardware. The PBX software generates small routines that copy one port's receive sample to another port's transmit sample, once per frame tick, and replaces them when a connection is made or cleared. 📖 Read; ✅ the copying was exercised in emulation.

## Extra ports

Ports 2 to 4 sit behind a serial control interface to an ISDN transceiver chip. At start-up the PBX reads the chip's identification register; if nothing answers, the port is disabled and the PBX continues without it. With a model that answers (an ISAC-SX type chip, identification value `01` on read), all four ports are configured. ✅ Tested. The line side of these chips (activation, HDLC, interrupts) is not modelled yet.

## The HDLC engine (D-channel)

The D-channel does not go through the sample registers. Each hardware channel (32 of them) has a small register block at 0x184000 + n × 0x40 and a ring of descriptors in RAM that the software owns and the hardware fills or empties. ✅ Tested, PBX-observed.

| Register | Meaning |
|---|---|
| +0x00 | address of the first **transmit** descriptor |
| +0x0c | transmit enable (the software writes 0 while setting up, 1 after queueing a frame) |
| +0x10 | address of the first **receive** descriptor |
| +0x1c | receive enable |

A descriptor is 16 bytes: `status`, `length` (low 16 bits), `buffer address`, `next descriptor`. In `status` bit 31 means *owned by hardware*, bit 30 *first fragment*, bit 29 *last fragment*.

- **Transmit:** the software fills the buffer, sets the length and then the status (`e0000000` for a single-descriptor frame). The hardware sends the frame, appends the FCS and clears the owner bit.
- **Receive:** the software leaves free descriptors with the owner bit set. The hardware writes the frame **plus a two-byte FCS** into the buffer and sets `status = 60000000 | length` (first, last, owner cleared; the length includes the FCS). The software strips the two bytes itself.
- Other status bits report errors (the software drops a frame with error bits set); the meaning of each error bit is 🔶 inferred.

The driver is **polled**: a task asks whether a received frame is ready and reads it. The completion callbacks it registers with the hardware layer only count events.

## Ethernet and management

The PBX accepts and sends private frames (EtherType 0x8888) that carry its internal messages to and from remote tasks; the same message format is used between the PBX and its management software. The framing and the default access are not documented here.

## What is not known

- The timer and interrupt-controller register layout of the main chip (the emulation models the periods).
- The flash layout outside the program image (boot block, configuration, database) and the update procedure.
- Power feed of the S0 ports, the analog and DECT module hardware, and the router module's CPU.
- Cycle timing: the emulation counts instructions, not bus cycles.
