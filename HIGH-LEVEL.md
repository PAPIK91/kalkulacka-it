# Kalkulačka služeb – přehled pro IT

Stav předávané verze: **26. 10. 2026**. Instalační postup je v [NASAZENI.txt](NASAZENI.txt), zdroje a hotový runtime v [kompletním balíčku](Kalkulacka-sluzeb-pro-IT.zip?raw=true).

## Účel a přístupy

Aplikace počítá cenu služeb pro zákazníka a připravuje textový zápis do případu (UZ).

| Část | Přístup |
|---|---|
| Kalkulačka | Veřejná bez přihlášení. Operátor zadává parametry a GOP, vidí rozpis a kopíruje UZ. |
| Správa ceníku | Určený správce se přihlašuje pouze heslem a upravuje cenové parametry. Server ověřuje každý zápis. |

Vstupy jednotlivých kalkulací a UZ se trvale neukládají. Operátor přenáší zápis do navazujícího systému.

## Hlavní funkce

| Region | Pravidla |
|---|---|
| CZ | Domácí ceník v CZK, jeho DPH a zahrnuté kilometry. |
| Praha, Brno | Samostatný městský ceník v CZK, vlastní tarify a kilometrový limit. |
| SK | Domácí ceník v EUR, jeho DPH a zahrnuté kilometry. |
| Zahraničí CZ | Ruční sazby; měna a DPH převzaté z CZ ceníku. |
| Zahraničí SK | Ruční sazby; měna a DPH převzaté z SK ceníku. |

Domácí služby: RMQ Standard, Nadrozměr, Hydraulika, DPV a Odemčení vozidla. Výpočet kombinuje paušál, kilometry nad zahrnutý limit a zvolené doplňky: vyproštění, manipulaci, čekání, úschovu, noc, víkend/svátek, vozíčky, brýle, přetah, mýto, jinou službu a volitelný palivový příplatek. Lze zadat celkovou trasu nebo přímo kilometry nad paušál.

V zahraničí operátor vybírá **paušál**, nebo **počet hodin × hodinovou sazbu**, a zadává vlastní sazbu za kilometr. Ruční sazby začínají na **0**. Účtuje se **celá trasa**; v paušálu nejsou zahrnuté kilometry. Jediným doplňkem je **Jiná služba** s vlastním názvem a cenou. Domácí příplatky ani palivo se neúčtují. Při změně měny se příslušné ruční částky resetují.

Výsledek obsahuje položky, cenu bez DPH, DPH, celkem a GOP. Doplatek vychází z nezáporného rozdílu ceny s DPH a GOP, násobeného nastaveným koeficientem (výchozí 1,20). Jeho řádek se zobrazuje v souhrnu i UZ **pouze při kladné částce**. UZ má víceřádkový formát ke kopírování do případu.

## Architektura a tok dat

| Komponenta | Úloha |
|---|---|
| React rozhraní | Kalkulačka, okamžitý přepočet, UZ a formulář správy. |
| Společné cenové jádro | Validace, výpočty a výchozí ceník; peníze počítá v celých haléřích/centech s definovaným zaokrouhlováním. |
| Node.js 24+ / API | Vydává rozhraní a ceník, přihlašuje správce, ověřuje a ukládá změny; nabízí také endpoint výpočtu. |
| SQLite | Trvalý ceník, verze, heslový hash a správcovské relace. |
| Docker + HTTPS proxy | Spuštění aplikace a přístup přes firemní doménu; přiložen je vzor Caddy. |

Prohlížeč načte ceník z API a přepočítává vstupy společným cenovým jádrem. UZ kopíruje do schránky. Správce se přihlásí přes HTTPS; server vytvoří časově omezenou relaci v HttpOnly cookie. Při ukládání ověří relaci, původ požadavku, hodnoty a verzi ceníku a změnu uloží do SQLite. Zastaralý souběžný zápis odmítne. Již otevřené jiné prohlížeče získají nový ceník po obnovení stránky.

Samostatný provoz nevyžaduje ChatGPT, Sites ani původní Jotform. Self-host varianta nahrazuje původní Cloudflare D1 lokální SQLite.

## Data, první nastavení a provoz

Data jsou v trvalém volume `/data`, databáze v `pricing.sqlite`. Výměna kontejneru musí volume zachovat. Databáze a zálohy obsahují také autentizační údaje a vyžadují omezený přístup. IT zajišťuje konzistentní zálohy a ověřenou obnovu. Dokumentovaný jednoduchý postup zálohuje celé volume během krátké odstávky, včetně případných SQLite WAL souborů; nestačí kopírovat jen živý hlavní soubor.

Balíček obsahuje výchozí ceník a obecné příklady konfigurace, **bez produkční databáze, hesel, hashů přístupů, tokenů a skutečných adres serveru**. Přenos upraveného provozního ceníku je samostatný krok.

IT nastaví doménu, DNS, HTTPS proxy, přesný `PUBLIC_ORIGIN`, úložiště a službu. Při první instalaci vznikne chráněný jednorázový `/data/setup-token`. IT jej bezpečně předá určenému správci, který nastaví vlastní heslo (12–256 znaků). Poté token zanikne a další přihlášení používá pouze heslo. Heslo je uložené jako solený PBKDF2 hash; relace používají hashované náhodné tokeny. Samoobslužné obnovení zapomenutého hesla není implementováno.

## Ověření a odpovědnosti

Předávaná verze má úspěšné sestavení a TypeScript kontrolu, **61 testů cenového jádra** a **18 kontrol Node/SQLite** při posledním sestavení. **21 HTTP kontrol** bylo úspěšných při předchozím ověření; po posledních změnách nebyl celý tento soubor znovu spuštěn. Doplnění tohoto dokumentu nezahrnuje nové testování ani nasazení.

**IT** odpovídá za infrastrukturu, HTTPS, dostupnost, zabezpečení dat, zálohy, obnovu a aktualizace. Před převzetím ověří na cílovém prostředí veřejný přístup, přihlášení, ukládání ceníku, zachování dat po restartu a referenční cenu.

**Vlastník aplikace a ceníku** schvaluje sazby a obchodní význam koeficientu doplatku, přetahu a zaokrouhlování. Změna výpočetních pravidel vyžaduje úpravu kódu a nové ověření.

Technické testy nenahrazují obchodní schválení ani kontrolu firemní infrastruktury. Ceník je veřejně dostupný přes API; heslo chrání jeho změny. Aplikace neřeší automatické měnové konverze, účetnictví, historii případů nebo integraci s případovým systémem.
