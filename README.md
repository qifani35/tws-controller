# TWS BLE Probe

A single-page BLE probe for **earbuds whose vendor never shipped a control app**.
Enumerate GATT services, subscribe to notifications, and send raw hex — all from
Chrome on Android.

Live: **https://qifani35.github.io/tws-controller/**

## Why a probe and not a "mode switcher"

Cheap TWS earbuds run on a handful of chipsets (BES, Jieli, Qualcomm, Actions,
plus vendor-custom firmware like Audio-Technica's). Many expose a writable GATT
characteristic that changes the listening mode — the vendor just ships no app.

The service/characteristic UUIDs and command bytes are **vendor-specific and
undocumented**. There is no universal "ANC on" command. So the tool is built to
discover, not to assume.

## Usage

1. **Connect** — press *Connect to earbuds* and pick the device from the chooser.
2. **Explore** — paste the service UUID(s), comma or newline separated, and press
   *Explore services*. Every characteristic is listed with its properties
   (`read` · `write` · `write-no-resp` · `notify` · `indicate`) plus per-characteristic
   buttons.
3. **Probe** — set a write target with *Use for write*, then send hex. `Read` and
   `Subscribe` reveal what the earbuds report.

## Finding the mode command

The reliable recipe:

1. Press **Subscribe** on every characteristic that supports `notify`/`indicate`.
2. Change the mode **by touching the earbuds**.
3. Read the bytes that arrive in the log — that is your model's encoding.
4. Replay exactly those bytes with **Send**.

Starting guesses worth trying: `01 00`, `01 01`, `01 02`, `02 00`. If writes
disconnect the earbuds, switch to *Write With Response* — some firmwares need it.

## Where to get the service UUID

Use [nRF Connect](https://play.google.com/store/apps/details?id=no.nordicsemi.android.mcp):
connect to the earbuds, tap a service, and the UUID is shown at the top. Copy it
into the *Services to explore* box.

A UUID such as `8a22ac7d-7566-4bf3-9cc7-8e33e8c40ada` is a vendor-custom service —
those are exactly the ones worth probing, because standard Bluetooth services do
not carry vendor mode switches.

## When nothing works

| Symptom | Cause |
|---|---|
| No writable characteristic | Firmware only accepts touch controls — no app can change modes |
| Only `notify`/`indicate` present | It reports state; the write path is elsewhere or absent |
| Writes succeed, mode unchanged | Wrong command bytes for this chipset |
| Disconnect on write | A handshake frame is required first, or use Write With Response |
| Service UUID not found | The UUID is wrong, or the device only exposes it after a pairing/handshake |

## Requirements

- Chrome or Edge on **Android** (Web Bluetooth is absent in Firefox and Safari)
- HTTPS or `localhost` — the GitHub Pages URL above satisfies this

## Privacy

No server, no analytics, no network calls after load. Everything stays in the page.

## License

MIT
