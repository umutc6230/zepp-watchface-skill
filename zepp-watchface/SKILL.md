---
name: zepp-watchface
description: >-
  Use when creating, redesigning or installing a watch face (kadran) for an Amazfit / Zepp OS watch
  (GTR 3, GTR 3 Pro, GTR 4, GTS, Balance, Active, Cheetah), when zeus-cli / zeus preview / QR install
  is involved, or when the Zepp app says "Yukleme basarisiz / tekrar deneyin / install failed" after
  scanning the QR, or a watch face needs AOD, analog hands, animated gears or step/heart data.
---

# Zepp OS kadran: tasarim, uretim, yukleme

## Ozet
Kadran = zeus projesi (`app.json` + `watchface/index.js` + `assets/<target>.r/`). Gorseller Python/Pillow ile **uretilir**, JS yalnizca yerel widget'lari dizer. Referans proje: `C:\Users\umuti\projects\amazfit-kadran` (8 kadran, ureticiler `tools/`, `DURUM.md`). Yeni kadrana oradan basla, sablon kopyalama.

**Once model:** Zepp > Profil'deki tam adi al. Zepp OS olmayanlar (ilk GTR, GTR 2/2e, GTS 2, Bip U vb.) zeus ile kadran ALMAZ - burada dur, kullaniciya soyle.

## Yukleme (Windows cmd + telefon; iPhone 13'te dogrulandi, Android menusu ayni)
1. Bir kez: `npm i -g @zeppos/zeus-cli` (kurmadan once tara), `zeus login` (telefondaki Zepp hesabiyla ayni).
2. Telefon (iOS/Android), bir kez: Zepp > Profil > Ayarlar > Hakkinda > Zepp simgesine **7 kez** dokun -> Gelistirici Modu.
3. Her kadran: `cd /d C:\Users\umuti\projects\<proje>\faces\<ad> && zeus preview` (cmd'de `~` calismaz).
4. QR: Zepp > Profil > **saat** > en alta kaydir > Gelistirici Modu > sag ust tara > Yukle. QR bayatlar; her denemede yeni komut.

**"Yukleme basarisiz / tekrar deneyin" -> once `zeus preview -s`.** zeus son secilen modeli hatirlar (`~/.zepp/.zeus` icinde `zeus_preview_device`) ve sormadan ona derler; saat farkli modelse Zepp reddeder. Kullanicidan Zepp'teki tam model adini al, `-s` ile sec. Ancak bu da olmazsa: Bluetooth/baglanti, Zepp onde acik, eski kadrani sil, `version.code` artir.

## app.json (tum yuvarlak modeller tek hedef)
```json
"configVersion": "v3",
"app": { "appId": 1093621, "appType": "watchface", "version": { "code": 2, "name": "1.0.1" }, "icon": "icon.png" },
"permissions": ["data:user.hd.step"],
"runtime": { "apiVersion": { "compatible": "1.0.0", "target": "1.0.1", "minVersion": "1.0.0" } },
"targets": { "common": { "module": { "watchface": { "path": "watchface/index", "main": 1 } },
  "platforms": [{ "st": "r" }], "designWidth": 480 } }
```
- Gorseller `assets/common.r/`; zeus `designWidth`'e gore cihaza olcekler (454/466/480 icin ayri klasor GEREKMEZ).
- `appId`: gelistirici onizlemesinde herhangi benzersiz sayi; her kadrana ayri.
- API 1.0 = GTR 3 ailesi dahil hepsinde calisir: global `hmUI`, `hmSensor`, `hmSetting`, `timer` (`@zos/*` import'lari API 2+).

## index.js kaliplari (API 1.0)
- Olcek: `const k = hmSetting.getDeviceInfo().width / 480; const s = v => Math.round(v * k)` - TUM koordinatlar `s()` ile. `px()`'e guvenme.
- AOD: `const aod = hmSetting.getScreenType() == hmSetting.screen_type.AOD`; `show_level = aod ? hmUI.show_level.ONAL_AOD : hmUI.show_level.ONLY_NORMAL` (ONAL yazimi dogru). AOD'de ayri `aod.png` + saniyesiz ibre + dis hatli ibre gorselleri.
- Katman sirasi = olusturma sirasi: zemin IMG en once, ibre (`TIME_POINTER`) sona, cam/yansima en ustte.
- Yerel widget'lar (timer gerektirmez, AOD'de calisir): `TIME_POINTER` (`hour_posX/posY` = ibre gorselindeki donme noktasi), `IMG_TIME`, `IMG_DATE` (`month_is_character` -> 12 gorsel), `IMG_WEEK` (7 gorsel, 1 = Pazartesi varsayimi; saatte dogrula), `TEXT_IMG` + `hmUI.data_type.STEP/HEART/BATTERY`.
- Hedefe gore ilerleme: `hmSensor.id.STEP` `.current/.target`, `addEventListener(hmSensor.event.CHANGE)`; segmentler `setProperty(hmUI.prop.VISIBLE)`, yay `ARC_PROGRESS` + `setProperty(hmUI.prop.MORE, tumParametreler)`.
- Animasyon: donen parca `IMG` + `setProperty(hmUI.prop.ANGLE, tamsayi)`, `timer.createTimer(0, ms, fn, {})`; `WIDGET_DELEGATE` `resume_call/pause_call` ile baslat/durdur. Kare animasyonu `IMG_ANIM` (`anim_path`, `anim_prefix`_i.png).

## Gorsel uretimi (Pillow)
- 4x supersampling ile ciz, LANCZOS ile kucult. Ibre gorseli ucu yukari, pivot `(W/2, L)`; golge eklenirse pivot pad kadar kayar.
- Yardimcilar referans projede: `gen_analog.py` (polar, steel_ring konik metal, ceramic, grain, glass, applied_index, hand sekilleri, aod_bg, aod_hand), `movement.py` (iskelet carklar, Cenevre kopru), `gen_lcd.py` (7/14 segment), `icons.py` (esit cizgi ikon), `audit.py` (gercek olcude normal+AOD sayfasi).
- Bitirmeden: `python tools/audit.py 466` uret, goruntuyu oku; tum `faces/*/watchface/index.js` icin `node --check`; her kadranda `zeus build`.

## Sik hatalar
| Belirti | Neden / cozum |
|---|---|
| QR sonrasi "tekrar deneyin" | Yanlis model -> `zeus preview -s` |
| cmd "yol bulunamiyor" | `~` yok -> `cd /d C:\Users\...` |
| Zemin saati/ibreyi kapatiyor | Widget sirasi; zemin once |
| Halka/daire dev cizildi | Yaricapi iki kez SS ile carpma |
| Kare koselerde doku/kar | grain'i daire maskesiyle uygula, disi siyah |
| Marka/logo | Kisisel kullanimda yazi olabilir, logo cizme, paylasma |
