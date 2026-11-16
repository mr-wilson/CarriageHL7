# CarriageHL7

CarriageHL7 sends and receives HL7 v2 over MLLP. Open a file, read it field by field, search it, compare two messages, check it against a site agreement, send, and answer what arrives.

The Windows edition is sold in the Microsoft Store for a nominal fee, and a purchase includes a 30-day trial. The Mac and Linux downloads are free. If there is enough interest, I will make CarriageHL7 available on the Apple App Store.

![Send. The message is on the left, the fields are on the right, and the send options are along the bottom.](images/send.png)

![Listen. Received messages stay in their own list, with the reply along the bottom.](images/listen.png)

[Get CarriageHL7 for Windows](https://apps.microsoft.com/detail/9MT80L709S60)

The Mac build is an Apple Silicon disk image. Open it and drag CarriageHL7 to Applications. macOS blocks the first launch because the app is not notarized. Control-click CarriageHL7, choose Open, then choose Open again.

[Download CarriageHL7 for Mac](https://github.com/mr-wilson/CarriageHL7/releases/latest/download/CarriageHL7.dmg)

The Linux build is an x86-64 tarball. Unpack it and run `./run.sh samples/09-search.hl7`. On Ubuntu, install the window libraries first: `libxkbcommon0` `libwayland-client0` `libx11-6` `libxcb1` `libxcursor1` `libxrandr2` `libxi6` `libegl1` `libgl1` `libvulkan1` `mesa-vulkan-drivers` `xvfb`. With no display, `run.sh` uses Xvfb.

[Download CarriageHL7 for Linux](https://github.com/mr-wilson/CarriageHL7/releases/download/linux-x64/carriage-linux-x64.tar.gz)

Publisher: Carriage · [Ryan Wilson on LinkedIn](https://www.linkedin.com/in/mrryanwilson/)

## Features

**Read.** Open a file, drop it on the window, or paste. Each segment is a row. The field list shows the path, a short name, and the value. Edits stay in memory. The file on disk is not changed.

**Send.** Send the current message, every message, or the next few. Wait for the acknowledgement, and stop after a reject, a timeout, or a control id that does not match. Save a route for the address, the wait, and TLS. Turn on a local log when you want one.

**Listen.** Received messages stay in their own list. Choose the acknowledgement, or leave Reply empty and the answer follows MSH-15 and MSH-16. The listener can answer at once or after a delay.

**Find.** Find narrows the list on the screen you are using. Send searches the file you opened. Listen searches the messages that arrived. Nothing is deleted. Previous and Next move through the matches.

A plain word is a case-insensitive match on the control id (MSH-10), message type (MSH-9), sending facility (MSH-4), patient identifier (PID-3), or patient name (PID-5). `Rivera` matches a name that contains Rivera. It does not search the rest of the message. For that, use `TEXT`.

Anything with a comparison is a small query language. Words such as `AND` and `CONTAINS` ignore case. A value can be bare, or quoted with `'` or `"` when it contains a space.

| You write | It means |
|---|---|
| `PID-5.1 = Rivera` | The family name is exactly Rivera. |
| `name contains Rivera` | PID-5 contains Rivera. The whole name `Rivera^Case` does not equal `Rivera`. |
| `MSH-9.2 = A03` | The trigger event is A03. |
| `type contains ADT` | MSH-9 contains ADT, so `ADT^A01` matches. `type = A03` does not, because the field is not only A03. |
| `mrn = PAT1 AND MSH-9.2 = A01` | Both are true. |
| `MSH-9.2 = A01 OR MSH-9.2 = A08` | Either is true. |
| `NOT MSH-9.2 = A08` | The trigger is not A08. |
| `PV1-2 = I` | Patient class is I. A segment id may contain digits, as in PV1, NK1, or OBX. |
| `PID-3[2].5 = MR` | The second identifier's type is MR. |
| `PID-7 EMPTY` | Birth date has no value. `EXISTS` is the opposite. Neither takes a value. |
| `OBX-5 < 100` | A numeric result is less than 100. |
| `PV1-44 > 2026` | Admit date is after the year 2026. |
| `TEXT contains note` | A field other than the encoding characters in MSH-1 and MSH-2 contains note. |
| `{Discharges} AND facility = MEM` | The saved search named Discharges, plus this facility. |

A path is `SEGMENT-field`, then an optional repetition in brackets, then `.component` and `.subcomponent`. `MSH-9.2` is the trigger. `PID-5.1` is the family name. You can also use the short name from the field list, such as `patient class = I`. A name shared by more than one field matches any of them. These shortcuts stand for one field: `control` is MSH-10, `type` is MSH-9, `facility` is MSH-4, `mrn` is PID-3, `name` is PID-5, and `ack` is MSA-1.

`=` is the whole value. `!=` matches when none of the values equal it. `CONTAINS` is a substring. `LIKE` is a substring until you add a wildcard: `%` is any run of characters and `_` is one character, as in `MSH-9.2 LIKE A0_`. Comparisons ignore case. `AND` binds more tightly than `OR`, and `NOT` binds more tightly than `AND`. Parentheses group a clause.

`<` and `>` compare numbers when both sides are numbers. On a date or time field they use the calendar. A year is the whole year, and a day is the whole day, so a timestamp in 2026 is not greater than `2026`. `2026-01-15` is the same day as `20260115`. `=` on a date is still the exact text, so `MSH-7 = 2026` does not match `20260115103000`.

Click a value in the message or in the field list and Find fills in that path. Type a space and pick `=`, `!=`, `<`, `>`, `contains`, `like`, `empty`, or `exists`. A finished comparison plus another click adds `AND` and the next path. An unfinished comparison, such as `PID-5 =`, leaves the list where it was.

Type a name and **Save** keeps the query in the folder you last clicked. **Folder** makes a folder. **Searches** opens the tree. Drag a row into a folder, or between rows to place it. Right-click for Use, Rename, Duplicate, Delete, New folder, and Save current Find. **Combine** inserts a saved search with the `AND` or `OR` you selected. `{Discharges}` follows that search if you edit it later. **Export** writes the tree to JSON. **Import** adds searches from a file, and skips a query that is already saved under any name. An empty list starts with a set of common searches. **Clear** empties Find.

**Diff.** Compare two messages field by field. Each side can be the message on Send, the message on Listen, a file, or a paste. Rows that are the same are left out. The comparison is the decoded text, and it is exact.

**Rules.** A rules file is a site agreement: which message types it covers, which paths must be filled, and which codes a path may contain. Check runs the saved file and adds those findings to the built-in check. The Rules screen opens or pastes its own sample. Drag a field onto Required or Allowed, drop MSH-9 on Type, then Save. Edits stay off Check until you save. Listen still answers from the acknowledgement you set. The rules file does not change that reply.

**Check and redact.** Check lists missing segments, unknown segments, and broken escapes. Redact copies one message with the identity fields emptied. The message on screen is unchanged.

TLS uses a certificate and key you choose, for the connection you open or the port you listen on.

## Support

Email [mr.ryanwilson@gmail.com](mailto:mr.ryanwilson@gmail.com).

## Privacy

The app does not create an account and does not send data to Carriage. Settings, saved searches, rules files, and the optional log stay on your computer. Messages go only to the address you choose.

[Privacy policy](PRIVACY.md)
