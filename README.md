# swarm-macro-converter
This simple HTML and JavaScript code will convert Roccat Swarm macros to be compatible with Turtle Beach Swarm II and vice versa

# Swarm ⇄ Swarm II Macro Converter

A single-file, client-side tool for converting macro export files between **Roccat Swarm** (v1) and **Turtle Beach Swarm II**. No installation, no server, no dependencies — open the HTML file in any browser and it works.

---

## Usage

### Swarm 1 → Swarm II

1. Open **Roccat Swarm** → Macro Manager
2. **Check mark** a folder or individual macro
3. Click the **Share icon** (Share Selected Macros) and save the `.dat` file
4. Open the converter, select **Swarm 1 → Swarm II**, and drop in the `.dat`
5. Download the converted `.dat`
6. Open **Turtle Beach Swarm II** → Macro Manager → **Import** → select the converted `.dat`

### Swarm II → Swarm 1

1. Open **Turtle Beach Swarm II** → Macro Manager
2. Click **Export Macros**, then **check mark** a folder or individual macro and save the `.dat`
3. Open the converter, select **Swarm II → Swarm 1**, and drop in the `.dat`
4. Download the converted `.dat`
5. Open **Roccat Swarm** → Macro Manager → click the **Download icon** (Load Macro Set) → select the converted `.dat`

---

## What gets preserved

- All key press and release events
- Per-event delays (millisecond precision)
- Macro names and group/folder name
- Playback mode (once, hold/while pressed, toggle)
- Mouse button events (left, right, middle, side buttons)

---

## Technical Details

### File Format

Both Roccat Swarm and Turtle Beach Swarm II use the **identical binary format** — `ROCCAT01`. This means conversion is a clean re-pack with no lossy translation.

#### File Structure

```
[File Header]
  Bytes 0–7:   Magic "ROCCAT01" (ASCII)
  Bytes 8–11:  Version, big-endian uint32 (value: 1)
  Bytes 12–15: Group name length in bytes, big-endian uint32
  Bytes 16–N:  Group name, UTF-16 big-endian
  Bytes N+1–2: Null terminator (2 bytes)
  Byte  N+3:   Padding (0x00)
  Byte  N+4:   Macro count
  Bytes N+5–6: Padding (0x00 0x00)

[Macro Blocks]  — one per macro, each exactly 16,475 bytes (0x405B)
  Bytes 0–2:   Marker 0x40 0x57 0x01
  Bytes 3–4:   Playback mode, little-endian uint16
                 0x0000 = once
                 0x0001 = hold (while pressed)
                 0x0002 = toggle
  Bytes 5–6:   Unknown field, always 0x0001
  Bytes 7–86:  Macro name, ASCII, null-terminated, zero-padded to 80 bytes
  Bytes 87–88: Event count, little-endian uint16
  Bytes 89+:   Event records (8 bytes each, see below)
  Remainder:   Zero-padded to fill the 16,475-byte block

[Event Record]  — 8 bytes
  Byte  0:     Key code (USB HID scan code)
  Byte  1:     Direction — 0x01 = key down, 0x02 = key up
  Bytes 2–3:   Padding (0x00 0x00)
  Bytes 4–7:   Delay in milliseconds, little-endian uint32
```

#### Key Codes

Keys are stored as standard [USB HID keyboard scan codes](https://usb.org/sites/default/files/hut1_5.pdf). Mouse buttons use Roccat-specific codes in the `0xF0–0xF4` range:

| Code   | Button        |
|--------|---------------|
| `0xF0` | Left click    |
| `0xF1` | Right click   |
| `0xF2` | Middle click  |
| `0xF3` | Side button 1 |
| `0xF4` | Side button 2 |

### How the Converter Works

1. **Parses** the source `.dat` — reads the file header to extract the group name, then scans for `0x40 0x57 0x01` markers to locate each macro block
2. **Extracts** each macro's name, playback mode, and event list (keycode + direction + delay per event)
3. **Re-packs** the data into a fresh `ROCCAT01` binary using the same fixed block layout, which both Swarm versions accept on import

All processing happens in the browser via the [File API](https://developer.mozilla.org/en-US/docs/Web/API/File_API) and [DataView](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView). No data leaves your machine.

### Implementation

- Single HTML file — no build step, no npm, no dependencies
- Zero external requests (fonts are system stack, no CDN)
- ~500 lines of vanilla JavaScript

---

## Discovered By

The `ROCCAT01` binary format was reverse-engineered by inspecting real export files from both applications. No official documentation exists.
