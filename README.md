# PLC-automaty
[Co dodělat ]: #
[nic ]: #

# Programovatelné automaty (PLC)


## Cíl
- **Kategorizovat a porovnat** programovatelné automaty (PLC) podle aplikační oblasti, konstrukčního provedení (kompaktní, modulární na DIN lištu, distribuované I/O ostrovy) a odlišit je od embedded PLC, iPC, NC a mikrokontrolérů (MCU).
- **Analyzovat vnitřní hardwarovou architekturu PLC** (CPU, paměťové oblasti Flash/RAM/Retain, procesní obraz vstupů a výstupů PAE/PAA, sběrnice) na bázi ekosystému **Tecomat Foxtrot / TC800** a navrhnout dimenzování napájení a ochranu výstupních obvodů.
- **Sestavit a navrhnout** funkční hardwarovou konfiguraci modulárního PLC z reálných katalogů a technické dokumentace firmy **Teco a.s.** (včetně rozšiřujících modulů DI/DO/AI/AO a komunikačních linek TCL2 / CIB) na základě technologické I/O bilance s uvážením projektové rezervy.
- **Provést systematickou diagnostiku a troubleshooting** logického automatu s využitím provozních LED indikátorů, měřicích postupů digitálním multimetrem i softwarových nástrojů vývojového prostředí **Teco Mosaic** (Inspektor, Watch, Force, systémový log).
- **Implementovat a optimalizovat** řídicí algoritmus v normovaných jazycích dle **ČSN EN 61131-3** v prostředí Mosaic (**ST – Strukturovaný text** se stavovým automatem a **LD – Liniové schéma**) s aplikací standardních funkčních bloků časovačů (TON, TOF).
- **Zhodnotit a obhájit** volbu řídicího systému z hlediska celkových nákladů na vlastnictví (**TCO**), životního cyklu, dlouhodobé dostupnosti servisu a zásad funkční bezpečnosti (**Safety**).

## Ověření cílů

Programovatelné automaty (PLC)

1) Základní vlastnosti, rozdělení: podle použití, velikosti a podle modulárnosti
2) Popis částí PLC
3) Základní diagnostika PLC
4) Programovací prostředí a programovací jazyky pro PLC

---


# 1. Rozdělení PLC podle oblasti nasazení

Doplňte do tabulky vhodnou kategorii a konkrétního zástupce z vybraných výrobců. Jako vzor poslouží vyplněný řádek pro automatizaci domů a budov.

| Oblast nasazení | Vhodná koncepce PLC (kompaktní / modulární / embedded) | Typický zástupce z portfolia Teco (nebo ekvivalent) | Klíčové technické parametry (sběrnice, I/O, napájení, krytí) |
|---|---|---|---|
| Automatizace domů a budov (HVAC, Smart Home) - VZOR | Kompaktní PLC na DIN lištu s integrovanou instalací sběrnic | Tecomat Foxtrot 2 (např. centrální jednotka CP-2091) | Dvě ethernetová rozhraní, rozšiřitelné I/O, napájení podle konkrétní varianty, možnost připojení dalších modulů |
| Automatizace jednoúčelových strojů a linek | Kompaktní nebo modulární PLC | Teco Tecomat Foxtrot 2 (např. CP-2091) | Digitální a analogové vstupy/výstupy, Ethernet, komunikace TCP/IP, podpora IEC 61131-3 |
| Rozsáhlé procesní řízení a velké energetické celky | Modulární PLC s možností rozšíření vstupů a výstupů | Siemens SIMATIC S7-1500 | PROFINET, Ethernet, rozšiřitelné I/O moduly, průmyslová komunikace, možnost redundantního řešení podle konfigurace |
| Strojní zařízení s vestavěnou elektronikou (Embedded) | Embedded PLC (vestavěné PLC) | Beckhoff TwinCAT nebo OEM řídicí jednotky | Procesor, paměť, digitální a analogové I/O, Ethernet, možnost real-time řízení |

---

# 2. PLC vs. Embedded PLC a další řídicí jednotky

- Jaký je zásadní rozdíl mezi standardním průmyslovým PLC v krabičce na DIN lištu a tzv. Embedded PLC? Co obecně vyjadřuje pojem embedded a v jakých dalších typech zařízení se s ním setkáváme?

  - **Vaše vysvětlení:**

    Standardní PLC je samostatná průmyslová řídicí jednotka, která se většinou montuje do elektrického rozvaděče na DIN lištu. Používá se pro řízení strojů, výrobních linek a technologických procesů.

    Embedded PLC je vestavěný řídicí systém, který je přímo součástí konkrétního zařízení nebo stroje. Bývá menší a je přizpůsoben konkrétní aplikaci.

    Pojem Embedded znamená vestavěný. Setkáváme se s ním například v automobilech, domácích spotřebičích, mobilních telefonech, průmyslových zařízeních a dalších elektronických systémech.

- Doplňte do přehledové tabulky význam zkratek a oblast jejich nasazení:

| Zkratka | Co zkratka znamená (anglicky / česky) | Pro jakou oblast řízení se primárně využívá | Typický operační / řídicí systém |
|---|---|---|---|
| IPC | Industrial PC / Průmyslový počítač | Vizualizace SCADA, řízení rozsáhlých technologických uzlů, softPLC | Windows IoT Enterprise, Linux, RTOS |
| NC / CNC | Numerical Control / Computer Numerical Control (Numerické řízení) | Řízení obráběcích strojů, soustruhů a frézek | CNC řídicí systém, např. Siemens SINUMERIK |
| MCU | Microcontroller Unit / Mikrokontrolér | Řízení jednoduchých zařízení, senzorů a motorů | Bare-metal (bez OS) nebo RTOS |
| SoC | System on a Chip / Systém na čipu | Vestavěné systémy, chytrá zařízení a průmyslová elektronika | Embedded Linux, RTOS nebo bare-metal |

---


# 2. Část PLC – paměti, procesní obraz a provozní vlastnosti

## Blokové schéma PLC

```text
              Rozhraní pro obslužné pracoviště /
                    nadřazený systém
                            |
                    Centrální jednotka
                     /             \
             Systémová          Uživatelská
               paměť              paměť
                    |
              Systémová sběrnice
                    |
   ------------------------------------------------
   |          |          |         |               |
 Binární    Analogové  Rychlé   Polohovací    Komunikační
 vstupy a   vstupy a   čítače    moduly         moduly
 výstupy    výstupy
   |
   |                 |                  |
 Záložní          Speciální       Rozhraní pro
 paměťový           moduly         vzdálené I/O
 modul
```

## 1. Vnitřní uspořádání a paměťové oblasti PLC

- Podle výše uvedeného blokového schématu a technické dokumentace PLC Teco doplňte funkci jednotlivých paměťových bloků v procesní jednotce:

  - **Programová paměť (Flash):**
    - Uchovává uživatelský program PLC a jeho konfiguraci.
    - Data zůstávají zachována i po vypnutí napájení.

  - **Datová paměť (RAM):**
    - Slouží k ukládání aktuálních hodnot proměnných a pracovních dat programu.
    - Obsah se používá během běhu PLC.

  - **Procesní obraz vstupů a výstupů:**
    - Obsahuje aktuální stav vstupů a hodnoty určené pro výstupy.
    - Umožňuje programu pracovat s hodnotami během jednoho pracovního cyklu.

  - **Systémová paměť:**
    - Obsahuje systémové informace, konfiguraci PLC a data potřebná pro jeho správnou funkci.

  - **Remanentní paměť (Retain):**
    - Uchovává vybrané důležité proměnné i po výpadku napájení.
    - Používá se například pro počítadla, nastavené hodnoty a provozní stavy.

## 2. Cyklus řízení PLC a procesní obraz

![Princip cyklu PLC](https://upload.wikimedia.org/wikipedia/commons/6/6e/PLC_cycle.svg)

- Popište 3 základní fáze jednoho pracovního cyklu PLC (Scan Cycle):

  - **Fáze 1 (Čtení vstupů):**
    - PLC načte aktuální stavy fyzických vstupů a uloží je do procesního obrazu vstupů.

  - **Fáze 2 (Zpracování programu):**
    - Procesor vykonává uživatelský program a podle vstupních hodnot vypočítává výsledky.

  - **Fáze 3 (Zápis výstupů):**
    - Vypočítané hodnoty se zapíší do procesního obrazu výstupů a následně na fyzické výstupy PLC.

- Proč procesor PLC nepřetržitě opakuje tyto cykly, místo aby pracoval přímo s fyzickými vstupy a výstupy?

  - **Vysvětlení:**
    - Cyklické zpracování zajišťuje pravidelnou aktualizaci vstupů a výstupů.
    - Program pracuje s jednotným obrazem vstupních a výstupních hodnot.
    - Zjednodušuje řízení stroje a zajišťuje předvídatelné chování systému.

## 3. Napájení, diagnostika a záložní odolnost

- Jaký je provozní napájecí rozsah centrálních jednotek a rozšiřujících modulů systému Tecomat Foxtrot?

  - Centrální jednotky Foxtrot 2 jsou napájeny podle konkrétní varianty, běžně napětím **24 V DC**.
  - Napájení jednotlivých rozšiřujících modulů se může lišit podle jejich typu.

- Co se stane při výpadku napájení? Uveďte 3 kroky:

  1. PLC přestane vykonávat uživatelský program.
  2. Běžná data v nechráněné paměti RAM mohou být ztracena.
  3. Program ve Flash paměti a zálohované remanentní hodnoty zůstávají zachovány.

- Jaký typ záložní paměti a jaká technologie se používá pro uchování důležitých proměnných?

  - Používá se remanentní paměť RETAIN a nevolatilní paměť Flash.
  - Konkrétní způsob uchování dat závisí na typu PLC a konfiguraci.

- Proč musí být paměť s důležitými daty (např. aktuální stav počítadel) zálohována?

  - Aby při výpadku napájení nedošlo ke ztrátě důležitých provozních údajů.
  - Po obnovení napájení může zařízení pokračovat s uloženými hodnotami.

## 4. Komunikační rozhraní a systémová sběrnice Teco

- Doplňte do tabulky parametry sběrnic používaných v systému Tecomat Foxtrot.

| Název sběrnice | Typická rychlost / kabeláž | K čemu daná sběrnice slouží | Maximální dosah / zakončení |
|---|---|---|---|
| TCL2 (Teco Communication Line 2) | Sériová systémová linka, rychlost 345 kb/s | Připojení rychlých PLC I/O modulů, operátorských panelů a komunikačních modulů | Zakončení na konci linky, rozsah větve podle konfigurace systému |
| CIB (Common Installation Bus) | Dvouvodičová sběrnice (data + napájení) | Připojení čidel, ovladačů, snímačů a akčních členů v budovách | Až 500 m na nejdelší větev, bez zakončovacích impedancí |
| Ethernet (ETH) | Ethernet 10/100 Mb/s, kabeláž UTP/STP | Komunikace s počítačem, vizualizací, nadřazeným systémem a dalšími PLC | Standardně až 100 m na jeden měděný ethernetový segment |

---

## 5. Použité zdroje

1. Teco – Přehled systému Foxtrot:
   https://www.tecomat.cz/produkty/foxtrot-prehled-systemu

2. Teco – Foxtrot 2, základní moduly a technické parametry:
   https://catalog.tecomat.cz/kategorie/foxtrot-2-zakladni-moduly

3. Teco – Rozšiřující PLC I/O moduly TCL2:
   https://www.tecomat.cz/produkty/foxtrot-prehled-systemu/rozsirujici-moduly-tcl2

4. Teco – TCL2 Master Expander:
   https://catalog.tecomat.cz/index.php/produkt/se-0140-expander-1x-master-systemove-sbernice-tcl2-pro-foxtrot-2

5. Teco – Technické parametry centrální jednotky CP-2005:
   https://catalog.tecomat.cz/index.php/produkt/cp-200511nsnn

6. IEC – Norma IEC 61131-3:
   https://webstore.iec.ch/en/publication/68533




