# Call processing

!!! info "Evidence tags"
    ✅ tested in emulation · 📖 read from code or tables, not exercised · 🔶 inferred · ❓ unknown · 🔌 hardware-verified (nothing yet). See the [overview](../index.md#evidence-tags).

How one call moves through the PBX software, from the S0 line to call control and back. The structure is 📖 read from the PBX software and ✅ confirmed by logging every message that passes the PBX's central router while an emulated phone lifted the handset, dialled and hung up. Names here are descriptive, not the software's own.

## Layers

| Layer | Role |
|---|---|
| Line driver and L1 task | Polls the HDLC receive ring, passes frames up, writes frames down, and manages line activation. It also manages the network sync source and the front-panel indicators, but **sends no message to the upper layers when the line goes up or down**: they learn that a port is usable when its first frame arrives |
| Data link (Q.921) | TEI management and the LAPD state machine, one instance per port class |
| Network signalling (Q.931) | One task per side (network and user). Contains the system-phone extensions: registration, FACILITY with codeset 7, the call contexts. Finds a call context by an identification token and creates one on request |
| Call control | The core: set-up, release, hold, transfer, conference, deflection, display, key information, plus the system-phone registration and status events |
| Bearer call control | Seizing trunks and channels, digit handling, and the interface to the other feature tasks |
| Feature tasks | Menus and display control (system phones and DECT handsets), supplementary services, call completion, hotel, charging, voicemail, SMS, CTI and remote links |

## One outgoing call (observed)

| Step | What passes the router |
|---|---|
| Frame arrives | L1 task → data link: *data indication* → network signalling |
| Call created | network signalling → call control: *establish request* (the SETUP content) |
| Seizure | call control → bearer call control: *seizure*, answered by *seizure acknowledge*; a *direct-access request* goes to a further feature task; call control sends *channel information* back to network signalling |
| Reply on the wire | network signalling → data link (*unit data request* for the broadcast, *data request* for the I-frame) → L1 task → HDLC engine |
| Each digit | data indication → *address information* → call control → *digits* → bearer call control → feature task |
| Hang-up | *release request* → call control → *disconnect* and *release* to bearer call control and the feature task → *released* to network signalling; a *logical-id clear* is broadcast to the feature tasks |

Every hop is its own message, so the internal structure of the PBX can be re-implemented around a handful of call events without reproducing its task layout.

## Observations that matter for a replacement

- The wire behaviour of the PBX towards the phone depends on the call context in network signalling, not on call control. A FACILITY or menu message for an unknown token is **dropped silently**.
- The system-phone menu mechanism keeps a table of sessions keyed by that token (a few hundred entries) and hands each message to the session's handler; phones and DECT handsets use different handlers.
- Call control, not the signalling layer, decides when a number is complete and where the call goes.
