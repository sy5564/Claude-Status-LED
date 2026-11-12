---
name: configled
description: Set or change the address of the Claude Status-LED and test the connection. Example: /configled 192.168.1.50
argument-hint: "<IP address or name of the LED>"
disable-model-invocation: true
allowed-tools: Bash(curl.exe *), Bash(curl *), Bash(claude plugin configure *)
---

# Set the LED address

The user wants to set the address of their Claude Status-LED (an ESP on the local network) for the `statuslight` plugin.

Address given: $ARGUMENTS

Reply in the user's language, short and friendly. Proceed as follows:

1. **Determine the address.** If no address is given above (empty, or still the placeholder with a dollar sign), ask for the IP address or name of the LED. The LED setup page shows both after the Wi-Fi setup, e.g. `192.168.XXX.XXX` or `claude-led.local`.
2. **Normalize it.** Add `http://` if missing. Remove a trailing `/` and any path (e.g. `/hook`, `/led`). Result: `http://<host>`.
3. **Test the connection** with `curl.exe -s -m 4 "<address>/led?s=done&sid=configled"` (on macOS/Linux use `curl` instead of `curl.exe`).
   - Response `ok`: the LED now briefly shows "Done". Tell the user.
   - No response: tell the user the LED is not reachable (same network? LED powered? address correct?) and ask whether to save it anyway.
4. **Save** with:
   `echo '{"led_url":"<address>"}' | claude plugin configure statuslight@led-statuslight --values-stdin`
   (keep the JSON in single quotes)
   If the command fails or `claude` is not found: ask the user to open `/plugin`, choose **Configure** for the **statuslight** plugin and enter the address there.
5. **Finish.** Tell the user that **Claude Code must be restarted** for the new address to take effect.
