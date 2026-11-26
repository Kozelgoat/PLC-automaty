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


# 3. Inženýrská konfigurace funkční sestavy PLC Teco a proudová bilance

## 1. Konfigurace komponent z katalogu Teco

| Pozice | Typ modulu / komponenta | Přesné typové označení Teco | Objednací číslo (Part No.) | Pokryté vstupy a výstupy |
|---|---|---|---|---|
| 1 | Spínaný napájecí zdroj 24 V DC na DIN lištu | Mean Well DR-120-24 | DR-120-24 | Napájení centrály a senzorů 24 V DC |
| 2 | Centrální jednotka (CPU) | CP-2007 | CP-2007.11NDNN | 14 univerzálních DI/AI, 11 reléových DO, 2 tranzistorové DO/PWM, Ethernet, TCL2 a CIB |
| 3 | Rozšiřující modul 1 (sběrnice TCL2) | IB-1301 | TXN 113 01 | 12 digitálních vstupů DI 24 V DC |
| 4 | Rozšiřující modul 2 (sběrnice TCL2) | OS-1401 | TXN 114 01 | 12 tranzistorových výstupů DO 24 V DC |
| 5 | Případný další submodul / zakončení | Zakončovací člen TCL2 | Dle provedení sběrnice | Zakončení sběrnice TCL2, pokud je vyžadováno |

**Poznámka:** Pro dosažení požadovaných 23 digitálních vstupů je potřeba ještě druhý modul IB-1301. Ten je zahrnut v následujících výpočtech. OS-1401 poskytuje 12 tranzistorových výstupů, z nichž 4 využijeme pro PWM.
 
## 2. Vyhodnocení I/O bilance a projektové rezervy

| Typ signálu | Požadavek linky | Celková kapacita navržené sestavy Teco | Rezerva (počet volných svorek) | Výsledná rezerva v % |
|---|---:|---:|---:|---:|
| Digitální vstupy (DI) | 23 | 31 | 8 | 34,8 % |
| Analogové vstupy (AI: 4–20 mA, PT) | 7 | 7 | 0 | 0 % |
| Tranzistorové výstupy (DO 6 = 4 PWM + 2 DO) | 6 | 14 | 8 | 133,3 % |
| Reléové výstupy (DO relé 230 V) | 3 | 11 | 8 | 266,7 % |

### Rozdělení vstupů a výstupů

- **Digitální vstupy:** 7 vstupů na CP-2007 a 2 × 12 vstupů na IB-1301. Celkem 31 DI.
- **Analogové vstupy:** 5 proudových vstupů 4–20 mA a 2 vstupy pro teplotní čidla Pt1000 na CP-2007.
- **PWM výstupy:** 4 výstupy na modulu OS-1401.
- **Standardní tranzistorové výstupy:** 2 výstupy na CP-2007.
- **Reléové výstupy:** 11 výstupů na CP-2007, využity budou 3.

Rezerva v procentech je vypočítána podle vzorce:

`Rezerva = (kapacita - požadavek) / požadavek × 100`

## 3. Proudová bilance a dimenzování napájecího zdroje 24 V DC

### Výpočet proudové spotřeby

**1. Vlastní příkon CPU a rozšiřujících modulů:**

- CP-2007: maximální příkon 10 W.
- 2 × IB-1301: 2 × 2,5 W = 5 W.
- OS-1401: maximální příkon 2 W.

Celkem:

`P = 10 + 5 + 2 = 17 W`

`I = P / U = 17 / 24 = 0,71 A`

**2. Odběr napájených snímačů DI:**

`I = 23 × 10 mA = 230 mA = 0,23 A`

**3. Odběr zátěží na tranzistorových výstupech se soudobostí ks = 0,7:**

`I = 6 × 0,5 × 0,7 = 2,1 A`

**4. Celkový špičkový proud:**

`Icelk = 0,71 + 0,23 + 2,1 = 3,04 A`

**5. Potřebný proud zdroje včetně rezervy 25 %:**

`Izdroje = 3,04 × 1,25 = 3,80 A`

### Výběr napájecího zdroje

- Zvolený zdroj: **Mean Well DR-120-24**
- Výstupní napětí: 24 V DC
- Výstupní proud: 5 A
- Výkon: 120 W
- Potřebný proud sestavy: přibližně 3,80 A.
- Rezerva zdroje: přibližně 31,6 % nad vypočtený potřebný proud.

Zdroj je dostačující a splňuje požadovanou výkonovou rezervu 25 %.

### Alternativní zdroj jiného výrobce

**Mean Well HDR-150-24**

- Výstupní napětí: 24 V DC.
- Výstupní proud: 6,5 A.
- Výkon: 156 W.
- Montáž na DIN lištu.

Také tento zdroj splňuje požadavky sestavy a poskytuje větší proudovou rezervu.

---

# 4. Provozní diagnostika PLC, měření multimetrem a troubleshooting v IDE Mosaic

*Časová dotace: 25–30 minut | Klasifikovaná inženýrská úloha na známky*

## 1. Diagnostika podle stavových kontrolek (LED)

| Signalizační prvek na PLC | Stav (barva / svit) | Provozní význam |
|---|---|---|
| RUN | Zelená, trvale svítí | PLC pracuje a vykonává uživatelský program. |
| HALT / STOP | Žlutá / oranžová, svítí | PLC je zastavené a program se nevykonává. |
| ERR / FAULT | Červená, bliká | PLC signalizuje chybu nebo poruchový stav. |
| ERR / FAULT | Červená, trvale svítí | Závažná chyba PLC, která může bránit správnému chodu programu. |
| ETH (Link/Act) | Zelená/žlutá, bliká | Probíhá síťová komunikace přes Ethernet. |

## 2. Troubleshooting v elektrickém zapojení pomocí digitálního multimetru

### A. Detekce tvrdého zkratu mezi +24 V DC a 0 V (GND)

- Multimetr nastavím na měření odporu (Ω) nebo kontinuity.
- Zařízení musí být odpojené od napájení.
- Černý hrot připojím na 0 V (GND) a červený na +24 V.
- Pokud multimetr ukazuje téměř 0 Ω, může se jednat o zkrat.

### B. Ověření sepnutých kontaktů výstupního relé DO

- Vypnu napájení PLC a ověřím beznapěťový stav.
- Multimetr nastavím na měření kontinuity nebo odporu.
- Hroty připojím na svorky kontaktu relé.
- Sepnutý kontakt má téměř 0 Ω, rozepnutý kontakt ukazuje OL.

### C. Diagnostika napájecího zdroje a napájení PLC (24 V DC)

- Multimetr nastavím na měření stejnosměrného napětí (V DC).
- Černý hrot připojím na 0 V a červený na +24 V.
- Správná hodnota je přibližně 24 V DC.
- Zkontroluji stabilitu napětí a případné kolísání.

### D. Diagnostika vstupů a výstupů PLC (DI/DO)

- Multimetr nastavím na měření stejnosměrného napětí (V DC).
- Černý hrot připojím na GND a červený na měřený vstup nebo výstup.
- Změřím napětí při vypnutém a zapnutém signálu.
- U běžného 24V digitálního signálu očekávám přibližně 0 V při logické 0 a 24 V při logické 1.
- Kontinuitu měřím pouze při vypnutém napájení.

### E. Měření napětí 230 V AC pomocí multimetru

- Multimetr nastavím na měření střídavého napětí (V AC).
- Měření provádí pouze vyučující nebo kvalifikovaná osoba.
- Měří se mezi fázovým vodičem L a nulovým vodičem N.
- Správná hodnota je přibližně 230 V AC.

**Jaké riziko z toho vyplývá při následné práci na zařízení?**

- Hrozí úraz elektrickým proudem.
- Nesprávné nastavení multimetru může způsobit jeho poškození nebo zkrat.
- Před další prací je nutné ověřit beznapěťový stav.

## 3. Diagnostika v programovacím prostředí Teco Mosaic

### Nástroj Watch / Sledování proměnných (Inspektor)

**Účel:**
- Umožňuje sledovat aktuální hodnoty proměnných v PLC.
- Pomáhá zjistit, zda vstupy a výstupy správně reagují.
- Usnadňuje hledání chyb v programu.

### Funkce Force (Vnucení hodnoty vstupu / výstupu)

**Co přesně funkce provede:**
- Ručně vnutí vstupu nebo výstupu určitou hodnotu (0 nebo 1), nezávisle na běžném řízení programu.

**Provozní a bezpečnostní riziko funkce Force: Proč je její použití během ostrého provozu s přítomností lidské obsluhy zakázáno?**

- Může způsobit nečekané spuštění stroje nebo pohyb motoru.
- Může obejít běžné řízení a způsobit nebezpečnou situaci.
- Může ohrozit obsluhu nebo poškodit zařízení.

### Systémový diagnostický log (Chybový protokol PLC v Mosaic)

**Jaké informace v něm technik vyčte:**

- Chybová hlášení PLC.
- Restart nebo výpadek CPU.
- Výpadky komunikace na sběrnici TCL2.
- Chyby programu, například dělení nulou.
- Čas a průběh vzniku jednotlivých chyb.
