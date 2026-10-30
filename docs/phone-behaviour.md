# Phone behaviour

!!! info "Evidence tags"
    ✅ tested in emulation · 📖 read from code or tables, not exercised · 🔶 inferred · ❓ unknown · 🔌 hardware-verified (nothing yet). See the [overview](index.md#evidence-tags).

Observations from running the phone in emulation, with a PBX that registers the phone and then stays idle. These tell an implementer which features need PBX involvement and which do not.

## Local features

Most function keys act on the phone's own menus and send **nothing** on the D-channel. Screens reached after registration:

| Screen | Content |
|---|---|
| Idle | date and time, *Info*, *Tel. drctry* (appears only after registration), *Quiet*, *VIP* |
| Info | *Auto. Accept on/off* |
| Functions | *Acoustic*, *Configuration*, *Setting*, *Program nos.* |
| Configuration | *inhibit*, *MSN*, *Display*, *Charge*, *Call-Parking*, *Service* |
| Setting | *Direct call*, *Contrast*, *Call rerout.*, *Date*, *Call wait.*, *AUTO accept* |
| Volume etc. | *Headset*, *Ack. tones*, *Speaker*, *Quiet*, *Call wait.*, *handset* |
| Loudness | handset and loudspeaker loudness bars |
| Memo | *Memo entry*, *New*, *VIP*, *direct* |
| Station guarding | *Ringing signal off* |
| Answering machine | "This function is not available. If you want to use this feature please ask your dealer" |
| Program | *Program dial number*, *Program VIP number* |
| Language | *Which language?* (Italiano, Espanol, Deutsch, English, Francaise, Nederlands) on a blank memory |

Typing a code on the keypad (for example *Call-Parking* then `55`) shows the digits on the display. Whether the phone then dials a feature code toward the PBX or handles it internally was not checked.

## Directory (*Tel. drctry*)

The phone's directory is **stored in the phone**. Evidence (all ✅ Tested, in emulation):

- The programming menu (*Functions*, *Program nos.*, *Program dial number*, *Tel. drctry*) offers "telephone book, 250 Entries avail., New". This works with no PBX registered and sends nothing on the D-channel.
- Creating an entry works entirely inside the phone: name entry (*Input telephone book*, "Name>", multi-tap letters on the digit keys, a confirm key), then "Number>", confirm again. The counter drops to "249 Entries avail." and the entry menu then offers *Sp.ton. call*, *Info*, *New*, *delete*, *List*. No D-channel traffic occurs at any step.
- The *Tel. drctry* line on the idle screen opens a search of this local book ("Make initial select.", "Name>"). With an empty book pressing it appears to do nothing, which is why a first test saw no effect. This also sends nothing to the PBX.
- The entry is not written to the phone's small serial EEPROM; the phone uses its flash for it (the emulator does not store flash writes, so entries do not persist there). 🔶 Inferred from the phone's service menu entries *Flash* and *EEPROM*.

What registration changes (✅ Tested): the idle *Tel. drctry* line and the *direct* entry of the *Program dial number* menu only exist after the PBX has accepted the registration. Why they depend on it is ❓ Unknown.

So the phone's own directory cannot be supplied from the PBX side through anything found so far. The ICT PBX has a separate central telephone book (📖 Read: about 1000 entries and a second list, reached through its configuration interface); no path from it to the system phone was found. An earlier version of these notes guessed that the phone's directory comes from the PBX; the tests above do not support that.

## Settings stored in the PBX

🔶 **Inferred:** several entries in the *Configuration* and *Setting* menus (*inhibit*, *MSN*, *Charge*, *Call-Parking*, *Call rerout.*, *Service*) are PBX features, so their values are probably kept by the PBX and changed through the system-phone channel. ❓ Unknown: no message was seen when these menus were opened, and nothing was changed in them in the tests.

## Keys that start a call

Two function keys behave like lifting the handset and send a plain SETUP identical to the one in [Calls](calls.md), which the PBX sees as an outgoing call. One of them also sends a DISCONNECT and RELEASE COMPLETE afterwards. Which physical key they are is ❓ Unknown: the labels on the real phone were not checked.

## Date and time

The date and time on screen come from the registration reply. Without the date/time element, the phone shows its default time (01.01.03 00:00).

## Warning screen

The phone shows `WARNING (1): Flash!` after some time in emulation, with or without a PBX. Its meaning is not known; it may be an artefact of the emulation (the emulated phone also resets itself after a fixed time).

## Hook and keys

The hook switch and key matrix are local inputs. They matter to the PBX only through the messages they cause (SETUP when the handset is lifted, INFORMATION for digits, DISCONNECT when it is replaced).
