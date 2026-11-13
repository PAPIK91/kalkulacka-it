# Kalkulačka služeb – předání pro IT

Aktualizované balíčky ze dne 13. 11. 2026 pro nasazení na vlastním serveru.

- [Stáhnout kompletní projekt a hotovou aplikaci](Kalkulacka-sluzeb-pro-IT.zip?raw=true)
- [Stáhnout pouze dokumentaci pro IT](Dokumentace-pro-IT.zip?raw=true)
- [Číst návod k nasazení](NASAZENI.txt)
- [Přehled aplikace a architektury pro IT](HIGH-LEVEL.md)

V levém menu je nová funkce **Převod měn a DPH**. Operátor zadá CZK nebo EUR. Částka v CZK se převede na EUR kurzem ČNB a poté se odečte slovenská DPH 23 %; u částky v EUR se odečte pouze DPH. Aplikace zobrazuje datum kurzu.

U všech domácích služeb je volba Marný výjezd se samostatným paušálem pro službu a region. Kilometry a další vybrané služby se účtují dál. Ceník spravuje správce heslem; změny jsou evidovány v historii.

Kalkulačka je veřejná. Balíčky neobsahují provozní databázi, hesla ani tokeny.

## Kontrolní součty SHA-256

```text
47046895f0f42abca493b1fedeb0db9cc728b5e1b96dfc07ba1374d9b2e8bef8  Kalkulacka-sluzeb-pro-IT.zip
48e71f66f1e32ae1e7d2530ee12caff6e4d72175f9513dcec0c677a7143698d0  Dokumentace-pro-IT.zip
a0bd79bf8d45c73bbd1c9fb2a4297b5ec15331f8a16de416d1a3ca49a1b1f571  NASAZENI.txt
```
