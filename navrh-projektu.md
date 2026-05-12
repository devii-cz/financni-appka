# Projekt

## 1. Účel aplikace
Cílem projektu je webová aplikace pro sledování osobních výdajů a příjmů.<br>
Slouží k lepšímu přehledu o financích nejen v Kč, ale také v mnoho dalších měn a vizualizaci toho, za jaké předvytvořené kategorie (jídlo, doprava, zábava atd.) je utraceno nejvíce peněz.<br>
Aplikace je navržena jako PWA, což umožňuje instalaci přímo na homescreen telefonu pro rychlé zaznamenávání výdajů.

## 2. Use-case

**Praktický příklad:** 
1. Uživatel zaplatí v obchodě nebo restauraci určitou částku.
2. Otevře aplikaci přes ikonu na ploše mobilního telefonu.
3. Ve formuláři zadá hodnotu (např. `-150Kč`/`-6€`), vybere kategorii `Jídlo` a potvrdí.
4. Aplikace přepočítá aktuální celkový zůstatek (případně převede cizí měny na Kč) a přidá novou položku do historie transakcí.
5. *Offline režim:* Pokud v danou chvíli není k dispozici internet, aplikace data dočasně uloží lokálně do LocalStorage a odešle je na server ihned po následném připojení.

**Use-case diagram:**<br>
<center>
Uživatel zadá data<br>⇣<br>
JavaScript zkontroluje připojení<br>⇣<br>
Zápis do databáze přes REST API<br>⇣<br>Frontend dynamicky překreslí seznam a graf
</center>

## 3. Struktura projektu
Zdrojové kódy jsou pro lepší přehlednost a údržbu rozděleny do logických celků:

* **`index.html`** - Hlavní šablona aplikace obsahující uživatelské rozhraní (formuláře pro zadávání, kontejner pro výpis historie a prostor pro graf).
* **`style.css`** - Vzhled aplikace se zajištění responzivity na menších displejích.
* **`script.js`** - Hlavní logika aplikace (zpracování událostí formuláře, validace uživatelských vstupů a dynamická úprava DOMu).
* **`api.js`** - Samostatný soubor vyhrazený pro komunikaci s backendem (Fetch API).
* **`manifest.json`** - Konfigurační soubor PWA definující název aplikace, barvy rozhraní a ikony pro mobily.
* **`sw.js`** - Service Worker běžící na pozadí, který zajišťuje cachování nezbytných souborů a částečnou funkčnost offline.

## 4. Princip fungování klíčových částí

**Zpracování dat (JavaScript a HTML/CSS)**<br>
Po načtení stránky provede aplikace async požadavek na server a stáhne historii transakcí. Pomocí JS jsou následně vygenerovány příslušné HTML elementy, které se vloží do stránky. Přidání nového záznamu okamžitě aktualizuje zobrazená data bez nutnosti obnovovat celou stránku.

**Využití LocalStorage**<br>
Local Storage  je v projektu využito ke dvěma účelům:
1. Uložení uživatelského nastavení - konkrétně volby mezi světlým a tmavým režimem (Dark mode). Nastavení tak zůstává zachováno i při dalším spuštění.
2. Zálohování (Drafts) – slouží jako dočasná paměť pro transakce vytvořené v momentě, kdy je zařízení offline.

**Integrace externího API**<br>
Pro zachování jednotných statistik v Kč aplikace využívá veřejné Frankfurter API.<br>Pokud uživatel zadá transakci v cizí měně (např. EUR nebo USD), aplikace před zápisem do databáze provede asynchronní dotaz (Fetch API) na aktuální kurz a zadanou částku automaticky převede. 

K tomuto účelu je využit endpoint pro přímý převod konkrétní částky:
* **Endpoint:**<br>
`GET https://api.frankfurter.app/latest?amount={castka}&from={puvodni_mena}&to=CZK`
* **Příklad z praxe:**<br>Pokud uživatel zadá výdaj ve výši `15 EUR`, aplikace odešle požadavek na `https://api.frankfurter.app/latest?amount=15&from=EUR&to=CZK`.<br>
Z vrácené JSON odpovědi se vezme výsledná hodnota v Kč a ta se následně uloží pomocí backendu.

## 5. Datový model a požadavky na REST API

Pro trvalé ukládání dat aplikace komunikuje s backendem, kterému odesílá informace ve formátu JSON (příklad těla požadavku: `{"amount": 150, "category": "Jídlo"}`). 

Aplikace bude využívat tyto endpointy:
* `GET /api/transactions` - Načtení všech záznamů při spuštění aplikace.
* `POST /api/transactions` - Vytvoření a uložení nové transakce po odeslání formuláře.
* `PATCH /api/transactions/:id` - Úprava dat (např. pokud se uživatel upsal u částky)
* `DELETE /api/transactions/:id` - Odstranění konkrétní transakce (např. v případě chybného zadání).
