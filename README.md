# Claude Status-LED

**English** | [Deutsch](README.de.md)

An LED on an ESP shows what [Claude Code](https://claude.com/claude-code) is doing right now: working, waiting for your approval, or done. You see it at a glance, even when the window is in the background.

- **Flash the firmware in your browser:** plug in the ESP, click a button, done. No development tools needed.
- **One-time Wi-Fi setup** from your phone. No credentials in any code.
- **Settings page in the browser** at `http://claude-led.local`: color, animation (incl. rainbow), speed and on/off per state, brightness, hardware, firmware update. English and German.
- **One firmware for all chips:** ESP32, ESP32-S2/S3/C3/C6, ESP8266. LED type and pins are configured in the browser.
- **No script on your PC:** Claude Code sends its events with `curl` straight to the LED, and the LED evaluates them itself.

> Unofficial fan project, not affiliated with Anthropic. Inspired by [Iwaide/clawd-heartbeat](https://github.com/Iwaide/clawd-heartbeat).

## States

| State | Meaning | Default display |
|---|---|---|
| Ready | Claude is waiting for your input | blue, solid |
| Working | Claude is thinking or using tools | orange, pulsing |
| Waiting for approval | Claude needs your permission | red, blinking |
| Done | The answer is ready | fade green ↔ cyan, back to "Ready" after 60 s |
| Error | The request failed | red, fast blinking |

For each state you can choose color, animation (solid, pulse, blink, two-color fade or **rainbow** through all colors) and speed. The **"Show"** checkbox turns individual states off.

System indicators: **purple pulsing** = setup mode · **purple fast blinking** = connecting to Wi-Fi · **purple slow blinking** = Wi-Fi lost, retrying · **orange, faster and faster** = button held for factory reset

## Hardware

Any ESP32 or ESP8266 board plus an LED:

| LED | Wiring |
|---|---|
| **Addressable** (WS2812B, SK6812, NeoPixel, also rings/strips), *recommended* | data pin, 5 V/3.3 V, GND |
| RGB LED with 3 legs (+ resistors) | 3 pins, "common anode" selectable |
| Single-color LED (e.g. the onboard LED) | 1 pin. States then differ only by animation. |

Boards with a built-in RGB LED work without soldering, e.g. **M5Stack Atom Lite**, ESP32-C3/S3/C6 DevKit or Waveshare ESP32-S3-Zero. The setup offers ready-made presets for them.

---

## 1. Install the firmware

1. Open the **flash page**: **https://sy5564.github.io/Claude-Status-LED/** (Chrome or Edge on a computer)
2. Plug in the ESP via USB, click **Install firmware** and choose the port.
3. The chip is detected automatically and the matching firmware is installed.

> No device found? Install the driver for the USB chip (usually CP210x or CH340). On boards with USB directly on the chip (e.g. ESP32-C3 SuperMini), hold **BOOT** while plugging in.

## 2. Set up Wi-Fi (once)

1. Power the ESP. If its LED is pre-configured correctly, it pulses **purple**.
2. Connect your phone to the network **`Claude-LED-XXXX`**. The setup page opens by itself, otherwise open `http://192.168.4.1`.
3. Choose your Wi-Fi and enter the password, optionally a device name and a static IP.
4. Under **LED hardware**, pick a board preset or enter the pins. **Test LED** shows a rainbow.
5. **Connect & save**. The page shows the IP address and the LED turns green.
6. Tap **Done**. The device restarts and is reachable at `http://claude-led.local`.

> 2.4 GHz networks only. If the Wi-Fi goes down later, the device simply keeps trying to reconnect. It does not fall back to setup mode.

## 3. Connect Claude Code

Claude Code reports every event to the LED through **hooks** using `curl` (`POST /hook`). Nothing runs on your PC.

### Option A: add the hooks to your settings (recommended)

Works everywhere, including the Claude desktop app. Copy the content of [`hooks/claude-settings-hooks.json`](hooks/claude-settings-hooks.json) into your Claude Code settings:

- Windows: `%USERPROFILE%\.claude\settings.json`
- macOS/Linux: `~/.claude/settings.json`

The easiest way is to let Claude Code do it:

> *Merge the hooks from `hooks/claude-settings-hooks.json` into my `~/.claude/settings.json` without changing existing entries.*

The hooks take effect immediately.

- **macOS/Linux:** replace `curl.exe` with `curl` in the commands.
- **Different device name:** if you gave the LED another name, adjust `claude-led.local` in the commands.

### Option B: as a plugin (Claude Code in the terminal)

```
/plugin marketplace add sy5564/Claude-Status-LED
```

```
/plugin install statuslight@led-statuslight
```

The plugin uses `http://claude-led.local` by default. Set a different address with `/statuslight:configled 192.168.178.50` or via `/plugin` → statuslight → Configure.

> In the Claude desktop app, a locally added plugin was registered but not fully installed in our tests, so its hooks did not run. Use option A there.

Use **only one** of the two options, otherwise every event arrives twice.

### Test

Send a message in Claude Code. The LED switches to "Working" and to "Done" at the end. On the settings page, **Status → Active sessions** lists every session that sends events.

### Which event triggers which state

| Claude Code hook | State |
|---|---|
| SessionStart, SessionEnd, PermissionDenied | Ready |
| UserPromptSubmit, PreToolUse, PostToolUse, PostToolUseFailure | Working |
| PermissionRequest | Waiting for approval |
| Stop | Done |
| StopFailure | Error |

Several Claude sessions are tracked separately. The most important state is shown: approval > error > done > working > ready. While an approval is pending, other tools (e.g. from subagents) do not override the red.

**Known limitations:** there is no event for a granted approval. The LED stays red until the approved tool has finished. After rejecting or pressing Ctrl+C, it stays red until your next input.

## 4. Settings page

Open `http://claude-led.local` (or the IP address) in your browser:

- **Status:** current state, active sessions, Wi-Fi signal, uptime, firmware
- **Brightness:** takes effect immediately
- **Display per state:** on/off, color, animation, speed, preview
- **Timing:** when "Done" returns to "Ready", auto-off after inactivity
- **LED hardware:** type, pins, number of LEDs, color order, button
- **Network:** device name, static IP, set up Wi-Fi again
- **Access protection:** optional PIN (user `admin`). `/hook` and `/led` stay reachable without a PIN.
- **Firmware update:** upload the matching `<chip>-update.bin` from the [flash page](https://sy5564.github.io/Claude-Status-LED/) or the [`flasher/bin`](flasher/bin) folder
- **System:** restart, factory reset

The page language follows your browser and can be switched with **EN | DE** in the top right.

**Button:** short press = LED test · hold for 10 seconds = factory reset. The chip's BOOT button is used by default (GPIO0, GPIO9 on C3/C6, GPIO39 on M5Atom).

## HTTP API

| Request | Description |
|---|---|
| `POST /hook` | Raw hook JSON from Claude Code (multipart upload or JSON body). Read in chunks, so even very large events are fine. Response: new state, `kept` or `ignored`. |
| `GET /led?s=<idle\|tool\|wait\|done\|err>&sid=<id>` | Set a state directly (compatible with clawd-heartbeat) |
| `GET /api/status` | Status as JSON |
| `GET/POST /api/settings` | Read/change settings (JSON, changed fields only) |
| `POST /api/preview` | e.g. `{"state":"wait"}` to show it for 6 s |
| `POST /api/update` | Firmware upload (multipart) |

```bash
curl "http://claude-led.local/led?s=wait"
```

## License

The firmware files, the flash page, the hooks template and the plugin may be used free of charge. The source code is not public. See [LICENSE](LICENSE) and [third-party notices](THIRD_PARTY_NOTICES.md).
