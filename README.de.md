# Claude Status-LED

[English](README.md) | **Deutsch**

Eine LED an einem ESP zeigt, was [Claude Code](https://claude.com/claude-code) gerade macht: ob es arbeitet, auf deine Freigabe wartet oder fertig ist. So siehst du es auf einen Blick, auch wenn das Fenster im Hintergrund liegt.

- **Firmware im Browser flashen:** ESP anstecken, Knopf drücken, fertig. Keine Entwicklungsumgebung nötig.
- **Einmaliges WLAN-Setup** über das Handy. Keine Zugangsdaten im Code.
- **Einstellungsseite im Browser** unter `http://claude-led.local`: Farbe, Animation (inkl. Regenbogen), Tempo und An/Aus pro Zustand, Helligkeit, Hardware, Firmware-Update. Auf Deutsch und Englisch.
- **Eine Firmware für alle Chips:** ESP32, ESP32-S2/S3/C3/C6, ESP8266. LED-Typ und Pins stellst du im Browser ein.
- **Kein Skript auf dem PC:** Claude Code schickt die Ereignisse per `curl` direkt an die LED, die LED wertet sie selbst aus.

> Inoffizielles Fanprojekt, nicht von Anthropic. Die Idee stammt von [Iwaide/clawd-heartbeat](https://github.com/Iwaide/clawd-heartbeat).

## Zustände

| Zustand | Bedeutung | Standard-Anzeige |
|---|---|---|
| Bereit | Claude wartet auf deine Eingabe | blau, dauerhaft |
| Arbeitet | Claude denkt oder nutzt Werkzeuge | orange, pulsierend |
| Wartet auf Freigabe | Claude braucht deine Erlaubnis | rot, blinkend |
| Fertig | Die Antwort ist fertig | Farbverlauf grün ↔ türkis, nach 60 s wieder „Bereit“ |
| Fehler | Die Anfrage ist fehlgeschlagen | rot, schnell blinkend |

Für jeden Zustand stellst du Farbe, Animation (dauerhaft, pulsierend, blinkend, Farbverlauf mit 2 Farben oder **Regenbogen** durch alle Farben) und Tempo ein. Mit der Checkbox **„Anzeigen“** schaltest du einzelne Zustände ab. Für **Fehler** stellst du einen Timer ein, nach dem die LED wieder auf „Bereit“ wechselt (0 = bleibt).

Systemanzeigen: **lila pulsierend** = Setup-Modus · **lila schnell blinkend** = verbindet mit WLAN · **lila langsam blinkend** = WLAN getrennt, versucht es weiter · **orange, immer schneller** = Knopf wird für den Reset gehalten

## Hardware

Jedes ESP32- oder ESP8266-Board, dazu eine LED:

| LED | Anschluss |
|---|---|
| **Adressierbar** (WS2812B, SK6812, NeoPixel, auch Ringe/Streifen), *empfohlen* | Daten-Pin, 5 V/3,3 V, GND |
| RGB-LED mit 3 Beinchen (+ Vorwiderstände) | 3 Pins, „gemeinsame Anode“ wählbar |
| Einfarbige LED (z. B. die Onboard-LED) | 1 Pin. Zustände unterscheiden sich dann nur über die Animation. |

Boards mit eingebauter RGB-LED funktionieren ohne Löten, z. B. **M5Stack Atom Lite**, ESP32-C3/S3/C6 DevKit oder Waveshare ESP32-S3-Zero. Für diese gibt es fertige Vorlagen im Setup.

---

## 1. Firmware aufspielen

1. Die **Flash-Seite** öffnen: **https://sy5564.github.io/Claude-Status-LED/** (Chrome oder Edge am Computer)
2. ESP per USB anstecken, **Firmware installieren** klicken und den Port wählen.
3. Der Chip wird automatisch erkannt und die passende Firmware aufgespielt.

> Wird kein Gerät gefunden: Treiber für den USB-Chip installieren (meist CP210x oder CH340). Bei Boards mit USB direkt am Chip (z. B. ESP32-C3 SuperMini) beim Einstecken **BOOT** gedrückt halten.

## 2. WLAN einrichten (einmalig)

1. ESP mit Strom versorgen. Ist die LED schon richtig vorbelegt, pulsiert sie **lila**.
2. Mit dem Handy das WLAN **`Claude-LED-XXXX`** verbinden. Die Einrichtungsseite öffnet sich von selbst, sonst `http://192.168.4.1` aufrufen.
3. WLAN wählen und Passwort eingeben, optional Gerätenamen und feste IP.
4. Unter **LED-Hardware** eine Board-Vorlage wählen oder die Pins eintragen. **LED testen** zeigt einen Regenbogen.
5. **Verbinden & speichern**. Die Seite zeigt die IP-Adresse, die LED leuchtet grün.
6. **Fertig** tippen. Das Gerät startet neu und ist unter `http://claude-led.local` erreichbar.

> Nur 2,4-GHz-WLANs. Fällt das WLAN später aus, versucht das Gerät weiter, sich zu verbinden. Ein Setup-Modus startet dann nicht.

## 3. Claude Code verbinden

Claude Code meldet jedes Ereignis über **Hooks** per `curl` an die LED (`POST /hook`). Auf dem PC läuft kein Skript.

### Variante A: Hooks in die Einstellungen eintragen (empfohlen)

Funktioniert überall, auch in der Claude-Desktop-App. Den Inhalt von [`hooks/claude-settings-hooks.json`](hooks/claude-settings-hooks.json) in deine Claude-Code-Einstellungen übernehmen:

- Windows: `%USERPROFILE%\.claude\settings.json`
- macOS/Linux: `~/.claude/settings.json`

Am einfachsten lässt du Claude Code das selbst machen:

> *Übernimm die Hooks aus `hooks/claude-settings-hooks.json` in meine `~/.claude/settings.json`, ohne bestehende Einträge zu verändern.*

Die Hooks wirken sofort.

- **macOS/Linux:** in den Befehlen `curl.exe` durch `curl` ersetzen.
- **Anderer Gerätename:** Hast du der LED einen anderen Namen gegeben, `claude-led.local` in den Befehlen anpassen.

### Variante B: als Plugin (Claude Code im Terminal)

```
/plugin marketplace add sy5564/Claude-Status-LED
```

```
/plugin install statuslight@led-statuslight
```

Das Plugin nutzt standardmäßig `http://claude-led.local`. Eine andere Adresse setzt du mit `/statuslight:configled 192.168.XXX.XXX` oder über `/plugin` → statuslight → Configure.

> In der Claude-Desktop-App wurde ein lokal eingetragenes Plugin bei uns registriert, aber nicht vollständig installiert. Die Hooks liefen nicht. Nutze dort Variante A.

Nutze **nur eine** der beiden Varianten, sonst kommt jedes Ereignis doppelt an.

## 4. Einstellungsseite

`http://claude-led.local` (oder die IP) im Browser öffnen:

- **Status:** aktueller Zustand, aktive Sessions, WLAN-Signal, Laufzeit, Firmware
- **Helligkeit:** wirkt sofort
- **Anzeige pro Zustand:** An/Aus, Farbe, Animation, Tempo, Vorschau, Timer für „Fehler“
- **Zeiten:** wann „Fertig“ zu „Bereit“ wird, Auto-Aus nach Inaktivität
- **LED-Hardware:** Typ, Pins, Anzahl, Farbreihenfolge, Knopf
- **Netzwerk:** Gerätename, feste IP, WLAN neu einrichten
- **Zugriffsschutz:** optionale PIN (Benutzer `admin`). `/hook` und `/led` bleiben ohne PIN erreichbar.
- **Firmware-Update:** passende `<chip>-update.bin` von der [Flash-Seite](https://sy5564.github.io/Claude-Status-LED/) oder aus dem Ordner [`flasher/bin`](flasher/bin) hochladen
- **System:** Neustart, Werkseinstellungen

Die Sprache der Seite richtet sich nach deinem Browser und lässt sich oben rechts mit **EN | DE** umschalten.

**Knopf:** kurz drücken = LED-Test · 10 Sekunden halten = Werkseinstellungen. Vorbelegt ist der BOOT-Knopf des Chips (GPIO0, bei C3/C6 GPIO9, M5Atom GPIO39).

## HTTP-API

| Aufruf | Beschreibung |
|---|---|
| `POST /hook` | Rohes Hook-JSON von Claude Code (multipart-Upload oder JSON-Body). Wird stückweise gelesen, auch sehr große Ereignisse sind kein Problem. Antwort: neuer Zustand, `kept` oder `ignored`. |
| `GET /led?s=<idle\|tool\|wait\|done\|err>&sid=<id>` | Zustand direkt setzen (kompatibel zu clawd-heartbeat) |
| `GET /api/status` | Status als JSON |
| `GET/POST /api/settings` | Einstellungen lesen/ändern (JSON, nur geänderte Felder) |
| `POST /api/preview` | z. B. `{"state":"wait"}` für 6 s anzeigen |
| `POST /api/update` | Firmware-Upload (multipart) |

```bash
curl "http://claude-led.local/led?s=wait"
```

## Lizenz

Die Firmware-Dateien, die Flash-Seite, die Hooks-Vorlage und das Plugin dürfen kostenlos genutzt werden. Der Quellcode ist nicht öffentlich. Siehe [LICENSE](LICENSE) (englisch) und [Hinweise zu Drittkomponenten](THIRD_PARTY_NOTICES.md).
