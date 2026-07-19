# Tempo signature — nastavení přes Gmail API

Plán, jak rozšířit `tempo-email-signature.html` o tlačítko "Nastavit v Gmailu", které podpis nahraje přímo přes API místo copy-pastu. Omezeno na doménu `tempo.ooo`.

## Architektura

- **Browser-side flow** — uživatel klikne tlačítko, projde Google OAuth v prohlížeči, JS zavolá `gmail.users.settings.sendAs.update` s vygenerovaným HTML podpisem.
- **Endpoint:** `PUT https://gmail.googleapis.com/gmail/v1/users/me/settings/sendAs/{sendAsEmail}`
- **OAuth scope:** `https://www.googleapis.com/auth/gmail.settings.basic`

### Háček s logem

Gmail v API podpisu nepřijme `data:image/...;base64` (strippuje se). Před nasazením musí být logo hostované na veřejné URL (GCS bucket — viz hint v `tempo-email-signature.html` ř. 267-275).

## Omezení na doménu tempo.ooo

Dvě vrstvy, které jdou kombinovat:

### 1. Internal OAuth client (Google to vyřeší)

OAuth consent screen nastavený jako **Internal User Type** — Google sám odmítne externí účty mimo Workspace organizaci tempo.ooo.

### 2. Doménový check v JS (skutečná pojistka)

Po přihlášení ověřit `hd` claim v ID tokenu:

```js
const payload = JSON.parse(atob(idToken.split('.')[1]));
if (payload.hd !== 'tempo.ooo') {
  alert('Jen pro tempo.ooo');
  return;
}
```

Volitelná třetí vrstva (`hd=tempo.ooo` parametr v OAuth requestu) slouží jen jako UX hint, není to bezpečnostní bariéra.

## Setup v Google Cloud

Co jde přes `gcloud` CLI:

```bash
gcloud projects create tempo-signature
gcloud config set project tempo-signature
gcloud services enable gmail.googleapis.com
gcloud iap oauth-brands create \
  --application_title="Tempo Signature" \
  --support_email=honza@tempo.ooo
gcloud iap oauth-clients create BRAND_ID \
  --display_name="Tempo Signature Web"
```

**Co `gcloud` neumí** (musí se doklikat v Console UI, ~2 min):
- Authorized JavaScript origins
- Authorized redirect URIs

`gcloud iap oauth-clients` vytváří client primárně pro Identity-Aware Proxy. Client ID/secret jdou použít i pro normální web OAuth, ale origins/redirects se přes CLI nedají nastavit.

Alternativa: Terraform `google_iap_client` resource — overkill pro jeden client.

## Odhad práce na kódu

50-80 řádků JS k tomu, co už je v `tempo-email-signature.html`:
- Google Identity Services script tag
- Tlačítko "Nastavit v Gmailu"
- OAuth flow (token client)
- Doménový check z ID tokenu
- `fetch` na Gmail API endpoint s vygenerovaným podpisem

## Otevřené otázky

- Kde bude stránka hostovaná? (Origin pro OAuth client.)
- Kam nahrát logo? (GCS bucket — název? veřejný přístup?)
- Existuje už nějaký Google Cloud projekt pod tempo.ooo, nebo zakládat nový?
