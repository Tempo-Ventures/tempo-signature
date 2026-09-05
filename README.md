# tempo-signature

Generátor e-mailového podpisu Tempo. Veřejně na GitHub Pages:
https://tempo-ventures.github.io/tempo-signature/

Tenhle repozitář je jediný zdroj podpisu. Interní přehled nástrojů
(tools.tempo.ooo, repo `tempo-tools`) si ho při buildu stahuje odsud,
takže úpravy se dělají jen tady.

Na `tools.tempo.ooo` běží stránka za Google přihlášením a umí podpis zapsat přímo
do Gmailu; na GitHub Pages se ta část skryje. Detaily a nastavení OAuth clientu:
`setup-gmail-api.md`.
