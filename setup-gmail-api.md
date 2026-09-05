# Nastavení podpisu přes Gmail API

Na `tools.tempo.ooo` má stránka tlačítko **Set up in Gmail**: zapíše podpis rovnou
do vybraných odesílacích adres uživatele. Na veřejné GitHub Pages verzi se tlačítko
nezobrazuje, tam zůstává jen kopírování do schránky.

## Jak to funguje

1. Stránka se zeptá `/oauth2/userinfo` (endpoint oauth2-proxy) na přihlášený účet.
   Když odpověď nepřijde, přihlašovací část stránky se vůbec nezobrazí.
2. Kliknutí na tlačítko vyžádá přes Google Identity Services token se scope
   `https://www.googleapis.com/auth/gmail.settings.basic`. Účet je předvybraný
   z bodu 1, takže se člověk znovu nepřihlašuje; poprvé jen odklikne oprávnění.
3. `GET users/me/settings/sendAs` vrátí adresy. Při jediné adrese se podpis zapíše
   rovnou, při více se nabídne výběr.
4. `PATCH users/me/settings/sendAs/{adresa}` zapíše pole `signature`.

Token žije jen v paměti stránky, nikam se neukládá.

## Co je potřeba v Google Cloud

Projekt `infrastructure-489410` (org tempo.ooo), consent screen Internal — restricted
scope `gmail.settings.basic` proto nepotřebuje ověření aplikace.

- Gmail API: zapnuté (`gcloud services enable gmail.googleapis.com --project=infrastructure-489410`).
- OAuth client: **sdílený s oauth2-proxy** (viz `tempo-infrastructure-management`,
  `docs/2026-08-10-ochrana-status-tempo-ooo.md`). V Console mu musí být mezi
  *Authorized JavaScript origins* přidáno `https://tools.tempo.ooo`. CLI to neumí —
  `gcloud iap oauth-clients` se vypíná 19. 3. 2026 a JS origins nenastavuje,
  `gcloud iam oauth-clients` je Workforce Identity Federation.
- Client ID patří do konstanty `GOOGLE_CLIENT_ID` v `index.html`. Je to veřejná
  hodnota, client secret stránka nepoužívá. Dokud je konstanta prázdná, tlačítko se
  nezobrazí.

## Co API neumí

Zaškrtávátko „Insert signature before the quoted text in replies" Gmail Settings API
nevystavuje. Zůstává v ručních instrukcích na stránce.
