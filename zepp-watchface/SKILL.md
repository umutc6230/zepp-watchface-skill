---
name: zepp-watchface
description: >-
  Use when creating, redesigning or installing a watch face for an Amazfit / Zepp OS watch
  (GTR 3, GTR 3 Pro, GTR 4, GTS, Balance, Active, Cheetah), when zeus-cli / zeus preview / QR install
  is involved, or when the Zepp app says "Install failed / try again" (Turkish: "Yukleme basarisiz /
  tekrar deneyin") after scanning the QR, or a watch face needs AOD, analog hands, animated gears or
  step/heart data.
---

# Zepp OS watch face: design, generation, install

## Summary
Watch face = zeus project (`app.json` + `watchface/index.js` + `assets/<target>.r/`). Images are **generated** with Python/Pillow; JS only lays out native widgets. If you already have a working face project, start from it instead of copying a template.

**Model first:** get the exact name from Zepp > Profile. Non-Zepp OS watches (original GTR, GTR 2/2e, GTS 2, Bip U, etc.) CANNOT take zeus watch faces - stop here and tell the user.

## Install (Windows cmd + phone; verified on iPhone 13, Android menu is the same)
1. Once: `npm i -g @zeppos/zeus-cli` (scan before installing), `zeus login` (same account as the Zepp app on the phone).
2. Phone (iOS/Android), once: Zepp > Profile > Settings > About > tap the Zepp logo **7 times** -> Developer Mode.
3. Each face: `cd /d C:\path\to\<project>\faces\<name> && zeus preview` (`~` does not work in cmd).
4. QR: Zepp > Profile > **watch** > scroll to bottom > Developer Mode > scan icon top right > Install. The QR expires; run the command again for each attempt.

**"Install failed / try again" -> first `zeus preview -s`.** zeus remembers the last selected model (`zeus_preview_device` in `~/.zepp/.zeus`) and builds for it without asking; if the watch is a different model, Zepp rejects it. Get the exact model name from Zepp and select it with `-s`. Only if that fails: Bluetooth/connection, Zepp open in the foreground, delete the old face, bump `version.code`.

## app.json (one target for all round models)
```json
"configVersion": "v3",
"app": { "appId": 1093621, "appType": "watchface", "version": { "code": 2, "name": "1.0.1" }, "icon": "icon.png" },
"permissions": ["data:user.hd.step"],
"runtime": { "apiVersion": { "compatible": "1.0.0", "target": "1.0.1", "minVersion": "1.0.0" } },
"targets": { "common": { "module": { "watchface": { "path": "watchface/index", "main": 1 } },
  "platforms": [{ "st": "r" }], "designWidth": 480 } }
```
- Images go in `assets/common.r/`; zeus scales to the device based on `designWidth` (NO separate folders needed for 454/466/480).
- `appId`: any unique number works for developer preview; use a different one per face.
- API 1.0 = runs on everything including the GTR 3 family: globals `hmUI`, `hmSensor`, `hmSetting`, `timer` (`@zos/*` imports are API 2+).

## index.js patterns (API 1.0)
- Scale: `const k = hmSetting.getDeviceInfo().width / 480; const s = v => Math.round(v * k)` - wrap ALL coordinates in `s()`. Do not rely on `px()`.
- AOD: `const aod = hmSetting.getScreenType() == hmSetting.screen_type.AOD`; `show_level = aod ? hmUI.show_level.ONAL_AOD : hmUI.show_level.ONLY_NORMAL` (the `ONAL` spelling is correct). In AOD use a separate `aod.png` + no seconds hand + outline hand images.
- Layer order = creation order: background IMG first, hands (`TIME_POINTER`) last, glass/reflection on top.
- Native widgets (no timer needed, work in AOD): `TIME_POINTER` (`hour_posX/posY` = pivot point in the hand image), `IMG_TIME`, `IMG_DATE` (`month_is_character` -> 12 images), `IMG_WEEK` (7 images, assumes 1 = Monday; verify on the watch), `TEXT_IMG` + `hmUI.data_type.STEP/HEART/BATTERY`.
- Progress toward goal: `hmSensor.id.STEP` `.current/.target`, `addEventListener(hmSensor.event.CHANGE)`; segments via `setProperty(hmUI.prop.VISIBLE)`, arc via `ARC_PROGRESS` + `setProperty(hmUI.prop.MORE, allParams)`.
- Animation: rotating part = `IMG` + `setProperty(hmUI.prop.ANGLE, integer)`, `timer.createTimer(0, ms, fn, {})`; start/stop with `WIDGET_DELEGATE` `resume_call/pause_call`. Frame animation: `IMG_ANIM` (`anim_path`, `anim_prefix`_i.png).

## Image generation (Pillow)
- Draw at 4x supersampling, downscale with LANCZOS. Hand image points up, pivot `(W/2, L)`; if you add a shadow, the pivot shifts by the padding.
- Useful helpers to write: polar placement, conic metal ring, ceramic, grain, glass, applied indices, hand shapes, AOD background and AOD hands; skeleton gears / bridges for movement faces; 7/14-segment digits for LCD faces; consistent line icons; an audit script that renders normal + AOD at real size.
- Before finishing: render the audit page (e.g. at 466 px) and look at it; `node --check` every `faces/*/watchface/index.js`; `zeus build` in every face.

## Common errors
| Symptom | Cause / fix |
|---|---|
| "Try again" after QR | Wrong model -> `zeus preview -s` |
| cmd "path not found" | No `~` -> `cd /d C:\Users\...` |
| Background covers time/hands | Widget order; background first |
| Ring/circle drawn huge | Radius multiplied by SS twice |
| Texture/noise in square corners | Apply grain with a circle mask, outside black |
| Brand/logo | Text is OK for personal use; do not draw logos, do not share |
