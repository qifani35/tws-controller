# TWS Mode Controller

A single-page BLE controller for **ODM TWS earbuds that ship without an app**.
Lets you try switching ANC / Transparency / Off from the browser, over Web
Bluetooth.

Live: https://qifani35.github.io/tws-controller/

## Why this needs a discovery step first

Cheap TWS earbuds are built on a handful of chipsets (BES, Jieli, Qualcomm,
Actions). Many of them *do* expose a writable GATT characteristic that changes
the ANC mode — the vendor just never shipped an app.

The problem: the service/characteristic UUIDs and the command bytes are
**vendor-specific and undocumented**. There is no universal "ANC on" command.
So the workflow is always: discover, then send.

## Usage

1. **Discover** — install [nRF Connect](https://play.google.com/store/apps/details?id=no.nordicsemi.android.mcp)
   on Android, connect to your earbuds, and look for a characteristic with
   `Write` or `Write Without Response`.
   Copy its **service UUID** and **characteristic UUID**.
2. **Connect** — open the page in Chrome on Android, paste both UUIDs, and
   press *Connect to earbuds*. If you leave the characteristic UUID empty the
   tool lists every characteristic on the service so you can pick one.
3. **Probe** — try the three mode buttons and watch the earbuds. The default
   `01 00 / 01 01 / 01 02` triple is a common guess, not a standard.

Settings are stored in `localStorage`, so UUIDs survive a reload.

## If nothing happens

In rough order of likelihood:

| Symptom | Cause |
|---|---|
| No writable characteristic in nRF Connect | Firmware only accepts touch controls — not controllable by any app |
| Characteristic is `Indicate`/`Notify` only | It reports state; the write goes elsewhere |
| Writes succeed but mode never changes | Wrong command bytes for your chipset |
| Earbuds disconnect on write | Some firmwares require a handshake/auth frame first |

A `Notify`/`Indicate` characteristic is still useful: subscribe to it in nRF
Connect, change the mode **by touch**, and watch which bytes the earbuds report.
That reveals the encoding for your model.

## Requirements

- Chrome or Edge on Android (Web Bluetooth is not available in Firefox or Safari)
- Served over HTTPS or `localhost` — Web Bluetooth refuses insecure origins.
  The GitHub Pages URL above satisfies this.

## Privacy

Everything happens in your browser. No server, no analytics, no network calls
after the page loads. UUIDs and commands stay in your own `localStorage`.

## License

MIT
