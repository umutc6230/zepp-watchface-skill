# zepp-watchface skill

A [Claude Code](https://claude.com/claude-code) / Agent Skill for designing, generating and installing watch faces on **Amazfit / Zepp OS** watches (GTR 3, GTR 3 Pro, GTR 4, GTS, Balance, Active, Cheetah) with `zeus-cli`.

Covers: `app.json` for all round models, API 1.0 `index.js` patterns (scaling, AOD, analog hands, step/heart data, animated gears), Pillow-based asset generation, QR install via Zepp developer mode, and common install errors ("Install failed / try again").

## Install

```bash
git clone https://github.com/palamut62/zepp-watchface-skill
cp -r zepp-watchface-skill/zepp-watchface ~/.claude/skills/
```

## Installing a generated watch face on your watch

Only Zepp OS watches can take faces built with `zeus`. Check the exact model name in Zepp > Profile first; the original GTR, GTR 2/2e, GTS 2, Bip U and other non-Zepp OS models are not supported.

### 1. One-time setup on the computer

```bash
npm i -g @zeppos/zeus-cli
zeus login        # sign in with the same account as the Zepp app on your phone
```

### 2. One-time: enable Developer Mode in the Zepp app (iOS and Android)

1. Open Zepp > **Profile** > **Settings** > **About**.
2. Tap the Zepp logo **7 times**. A "Developer Mode" message appears.

### 3. Install each face

1. In a terminal, go to the face folder and start the preview:
   ```bash
   cd path/to/faces/<face-name>
   zeus preview
   ```
   On Windows `cmd`, use `cd /d C:\full\path` (`~` does not work). The first time, pick your watch model from the list.
2. A QR code appears in the terminal.
3. On the phone: Zepp > **Profile** > your **watch** > scroll to the bottom > **Developer Mode** > tap the scan icon (top right) > scan the QR > **Install**.
4. Keep the Zepp app open in the foreground and the watch connected over Bluetooth until it finishes.

The QR code expires, so run `zeus preview` again for each new attempt.

### If Zepp says "Install failed / try again"

- Run `zeus preview -s` and select the **exact** model shown in Zepp. zeus remembers the last model and silently builds for it; if your watch is a different model, Zepp rejects the face.
- If it still fails: check the Bluetooth connection, keep Zepp in the foreground, remove the old copy of the face from the watch, and increase `version.code` in `app.json`.

## License

MIT

---

Product Owner: Umut Çelik - [X](https://x.com/palamut62) - [GitHub](https://github.com/palamut62)
