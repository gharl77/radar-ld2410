# Template: LD2410C presence radar

**English** · [Français](LISEZ-MOI.md)

`radar-ld2410.yaml` is a copy of `presence-soussol.yaml` where everything
specific to the device is replaced by `NOM`. The rest (filters, engineering
mode keep-alive, Bluetooth proxy, diagnostics) is identical.

## Create a new radar

1. Copy `radar-ld2410.yaml` into ESPHome Device Builder as
   `presence-<room>.yaml`.
2. Replace `NOM` everywhere (5 places): in the `substitutions` block and in
   the three secret names.
3. Create the three secrets in ESPHome (Secrets, top right):
   - `presence_<room>_api_key`: 32 bytes in base64 (`openssl rand -base64 32`)
   - `presence_<room>_ota_password`
   - `presence_<room>_ap_password` (8 characters minimum)
4. Check the pins. `GPIO1/GPIO3` is the USB serial port: the first cable flash
   must be done with the radar unplugged. On a new build, prefer `GPIO17` (TX)
   and `GPIO16` (RX), and remove `baud_rate: 0` from `logger`.
5. First flash over USB, then everything goes over the air.

## Why a full file instead of a package

Secret names (`!secret ...`) cannot come from a substitution, so a shared
package would still need per-device edits. A self-contained file is the
simplest thing to maintain.

## Keeping the template in sync

After changing `presence-soussol.yaml`, regenerate the template: replace
`presence-soussol` / `Presence-Soussol` / `presence_soussol_` with their `NOM`
equivalents.
