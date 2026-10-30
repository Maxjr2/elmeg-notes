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

- ✅ **Tested:** the entry *Tel. drctry* is **not** on the idle screen before the PBX has accepted the registration. It appears as soon as the registration reply is accepted (it also appears in the *Program dial number* menu).
- 🔶 **Inferred:** because the entry depends on the PBX, the directory contents probably live in the PBX and the phone fetches them over the system-phone channel (codeset 7, probably the data blob element `02`). This matches a recollection from a user of the phone that the names are not stored in the phone itself; that recollection is not verified.
- ❓ **Unknown:** the request message and the format of the entries. In emulation none of the 22 function keys sent a directory request from the idle screen (only the two call-starting keys produced traffic), the key that opens the directory could not be identified, and no directory data was ever supplied. The answer needs a capture of a real PBX serving a CS410, see [Open questions](open-questions.md).

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
