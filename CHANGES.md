# Before deploy: placeholders to replace

Ads start **Monday 19 Oct 2026**. Find everything left with:

```bash
grep -rn "PLACEHOLDER\|WHATSAPP_NUMBER\|LINKEDIN_\|VOICE_TRANSCRIPTION\|TODO(Esteban)" --include=*.html .
```

| Placeholder | Where | What goes there |
|---|---|---|
| `LINKEDIN_CONVERSION_ID` | `encuesta2026.html`: `CONFIG` | Conversion ID from Campaign Manager. Until it's a number the click isn't tracked (the button still works) |
| `VOICE_TRANSCRIPTION_VENDOR` | `privacidad.html`, `privacy.html` (shown in the table) | Name of the transcription provider. If it isn't in the US, change the "Dónde"/"Location" cell too |
| `TODO(Esteban)` | `privacidad.html`, `privacy.html`, section 10 | Have a lawyer review it for MX, CL, PE, AR and Central America |
| `assets/images/og-saman.png` | all pages | A simple logo-on-background image is already there. Replace it if you like (keep 1200×630) |

## Check after deploy

- Run `https://samantech.com.co/encuesta2026?src=CO-AD-01` through the LinkedIn Post Inspector
  (linkedin.com/post-inspector) to clear its cache and confirm the title and image.
- On iOS Safari, Android Chrome and the LinkedIn app's in-app browser, tap the button and confirm
  WhatsApp opens with `Hola, quiero participar en el estudio (CO-AD-01)`.
- `/encuesta2026/` (with the trailing slash) should redirect to `/encuesta2026` and keep `?src=`.
- Lighthouse mobile on `/` and `/encuesta2026`.

## Ad links

`https://samantech.com.co/encuesta2026?src=<PAÍS>-<CANAL>-01`, where PAÍS is `CO MX CL PE AR CA`
and CANAL is `AD` (LinkedIn ad) or `DM` (direct message). Anything else falls back to `WEB-01`.
