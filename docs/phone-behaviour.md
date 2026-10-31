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
- The entry is not written to the phone's small serial EEPROM; the phone uses its flash for it (entries live in the flash data store described under [Start-up](#start-up-and-the-warning-screen)). 🔶 Inferred from the phone's service menu entries *Flash* and *EEPROM*.

What registration changes (✅ Tested): the idle *Tel. drctry* line and the *direct* entry of the *Program dial number* menu only exist after the PBX has accepted the registration. Why they depend on it is ❓ Unknown.

So the phone's own directory cannot be supplied from the PBX side through anything found so far. The ICT PBX has a separate central telephone book (📖 Read: about 1000 entries and a second list, reached through its configuration interface); no path from it to the system phone was found. An earlier version of these notes guessed that the phone's directory comes from the PBX; the tests above do not support that.

## Settings stored in the PBX

🔶 **Inferred:** several entries in the *Configuration* and *Setting* menus (*inhibit*, *MSN*, *Charge*, *Call-Parking*, *Call rerout.*, *Service*) are PBX features, so their values are probably kept by the PBX and changed through the system-phone channel. ❓ Unknown: no message was seen when these menus were opened, and nothing was changed in them in the tests.

## Keys that start a call

Two function keys behave like lifting the handset and send a plain SETUP identical to the one in [Calls](calls.md), which the PBX sees as an outgoing call. One of them also sends a DISCONNECT and RELEASE COMPLETE afterwards. Which physical key they are is ❓ Unknown: the labels on the real phone were not checked.

## Date and time

The date and time on screen come from the registration reply. Without the date/time element, the phone shows its default time (01.01.03 00:00).

## Start-up and the warning screen

✅ Tested in emulation. The phone keeps its settings in a flash data store made of two alternating blocks. A block is valid only if its first byte holds a marker value; the blocks live outside the downloadable firmware image, so a phone with an erased store has no marker.

- With no valid block the phone shows `WARNING (1): Flash!` and then **restarts itself**, again and again. It does not depend on the PBX or on any plug-in module (an earlier version of these notes guessed otherwise).
- Once the blocks are initialised (as a factory-fresh unit would have them) the warning never appears, the phone stays up indefinitely, and registration, polling and calls work without further restarts.
- On the first start with a blank memory the phone asks for the language (*Which language?*), then reaches the idle screen. It registers to the PBX only after that.

An implementer testing with a real phone should therefore expect the language prompt once and no warnings; a recurring *Flash* warning points at the phone's store, not at the PBX.

## Hook and keys

The hook switch and key matrix are local inputs. They matter to the PBX only through the messages they cause (SETUP when the handset is lifted, INFORMATION for digits, DISCONNECT when it is replaced).
