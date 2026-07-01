# Zadanie pre GNSS Geret v2025

Dokument popisuje zmeny/upravy v elektronike gnss-geret z 2024 tak aby vznikla
jednodoskova prenostitelna verzia.

## Strucny opis zmien

Cielom je dosiahnut vyssiu integraciu, mensie rozmery a odstranit nedostatky
z predchadzajucej verzie.

Hlavne zmeny:

* \[x] Z troch DPS zredukovat len na jednu dosku.
* \[X] Povodnu hotovu procesorovu dosku FireBeetle2 nahradit samostatnymi obvodmi
  priamo na DPS a usetrit tym miesto:

  * \[x] Pouzit rovnaky ESP32-WROOM-32E modul
  * \[x] USB/UART prevodnik pre programovanie a ladenie ESP32 aj s DTR+RST pre programovanie
  * \[x] Menic napatia z baterie na 3V3
  * \[x] Nabijaci obvod pre bateriu (lepsi ako na FireBeetle2)

* \[x] Doplnit flash pre zaznam dat.
* \[ ] Lepsia citatelnost displeja s casom.

Vacsina suciastok zostava zachovana, zhodna s verziou v2024.



## Nove suciastky

### ESP32

Zvolit modul:

* ESP32-WROVER-E-N16R8 - trosku vacsi rozmer, ale ma vacsiu pamat, vyvody len z dvoch stran.
* \[x] ESP32-WROOM-32E-N16R2 - mensi rozmer, mensia pamat, vyvody z troch stran.



Umiestnit antenou ku okraju DPS. Pod vstavanou antenou je potrebne odstranit
oblast zeme v jednotlivych vrstvach DPS tak aby nedoslo ku zatieneniu anteny (napr.
vrstvou GND, vid. keepout zone v datasheete). Idealne vyfrezovat z plosaku.

Doska WROVER sa pripajkuje priamo na hlavny DPS.

Nerozpustat obsah WROOM modulu priamo na hlavny DPS, odovodnenie:

* nepredpokladam, ze by sa podarilo dosiahnut vasiu mieru integracie ako samotnemu
  vyrobcovi Espressif - t.j. neusetri sa miesto na DPS
* rucne osadzovanie by bolo skoro nemozne
* vyzadovalo by to ozivit zapojenie aj samotneho procesora
* neskor by ake kolvek chyby mohli mat pricinu bud v osadeni alebo navrhu zapojenia
  procesora



### USB/UART prevodnik

Pouzit niektory z casto pouzivanych USB na UART prevodnikov pre ESP procesory.
Napr. CH340K (bol pouzity aj na FireBeetle2) alebo CP2104, ...

Dolezite je pripojit aj DTR a RTS na GPIO0 a RST aby sa dal procesor uviest do
programovacieho rezimu cez USB.

Inspirovat sa zapojenim podla schemy ku FireBeetle2 alebo napr. ku LiLyGo T-Call.



### SPI Flash

Zapracovat "velku" SPI flash, ktora sa bude pouzivat na zaznam udajov z letu.
Napr. 128Mbit (16MB) [Winbond W25Q128FV](https://www.winbond.com/hq/product/code-storage-flash-memory/serial-nor-flash/?__locale=en&partNo=W25Q128FV).
Pripojit na SPI rozhranie.

* \[x] TODO: vyspecifikovat, na ktore piny procesora pripojit. Ktore z dvoch SPI rozhrani je volne?



### Nabijaci resp. power-management modul

Na napajanie bude pouzity jeden clanok Li-Ion baterie (v letectve sa nepouzivaju Li-Pol).
Mal by obsahovat:

* \[x] moznost vycitat cez komunikaciu (idealne I2C) stav nabijania (miera vybita)
* \[x] coulomb counter na meranie kapacity bateria
* \[x] meranie napatia baterie

Zapracovat modul na sledovanie kapacity LiXy batetie MAX17043.
https://techfun.sk/produkt/max17043-i2c-modul-pre-sledovanie-stavu-baterie-1s/

https://www.sparkfun.com/mikroe-charger-2-click.html
Coulomb counter TC3100 + nabijaci obvod IP5306-I2C.
https://electronics.stackexchange.com/questions/17463/help-on-stc3100-calculations
https://github.com/danirebollo/ArduinoSTC3100

* \[x] TODO: upresnit.



### Tlacidlo

Pridat nove mikro-tlacidlo niekam na pravu prednu stranu.

* \[x] TODO: Vyspecifikovat tlacidlo, umiestnenie a pripojenie.



### Indikacna LED

Pridat SMD modru LED na stranu DPS s displejmi. Pouzije sa na zakladnu indikaciu
a ladenie SW. Spolu s LED na rovnaky ovladaci pin procesora pripojit aj test-pad.
Nakolko su vsetky ostatne LED pripojene len cez komunikacne rozhranie, tato bude
urcite fungovat vzdy.
Zapojit na GPIOxy voci GND.

* \[x] TODO: Urcit, na ktory GPIO pin.



### Senzor osvetlenia

Doplnit senzor osvetlenia a automaticky upravovat jas.
Umiestnit ho do vnutorneho rohu kde sa stretava maly dolny displej a stredny velky displej
na pravej strane (na opacnej ako je encoder, aby sa rukou nezakryl).
V kryte sa na danom mieste urobi vybratie, cez ktore (a cez foliu) bude prenikat okolite svetlo
az ku DPS.

* \[x] TODO: Pouzit jednoduchy fototranzistor? Vyspecifikovat.



## Upravy

Vo verzii v2024 boli na DPS/scheme identifikovane nesledujuce body na zlepsenie:

* \[x] Doplnit USB iface pre gnss (3v3 urovne, pripojit vdd\_usb)
  Zvazit umiestnenie plnohodnotneho USB micro-B konektora otoceneho ku okraju DPS
  kvoli servisnemu pristupu do GNSS modulu. Pouzit uhlovy konektor o 90st zarovnany
  s okrajom DPS. Pristupny bude len po vybrati DPS z krabicky.
* \[x] Pridat odrusovaci kondik na tlacidlo tocitka.

  * TODO: Vyspecifikovat.

* \[x] Na vstup A a B z encodra doplnit pull-up rezistory 10k na 3V3.
* \[x] Do footprintu encodera pridat aj diery pre upevnovacie otvory (pajkovatelne, spojene s GND)
* \[x] Pouzit iny typ stredneho 4-ciferneho displeja:

  * Na v2024 bol pouzity typ so spolocnou anodou (KW4-804AGB).

    * Radic HT16K33 je koncipovany tak, ze:
    * cez samostatne anody dokaze do LED pustit 20mA
    * ces spolocnu katodu dokaze preniest 8x 20mA
    * kvoli tomu nebolo mozne vybudit LED na maximalny jas co sa prejavilo na zlej
      citatelnosti za slnecneho svetla.

  * \[x] Vo v2025 pouzit model so spolocnou katodou (KW4-804CGB).

    * Tomu sa musi prisposobit aj zapojenie jednotlivych segmentov (po vzore maleho displeja).
    * \[x] TODO: vyspecifikovat zapojenie displeja.

* \[ ] Pouzit USB-C konektor, zapojenie rovnake ako na FireBeetle2.
* \[x] Ku GNSS modulu pripojit aj zalohovaciu bateriu pre RTC.



### Chybajuce veci

Vo verzii v2024 chybali nasledujuce veci, ktore su aplikovatelne aj na v2025:

* \[x] Na 5V vetvu pridat velky stabilizacny kondik 470u/16V.
* Doplnit oznacenie D2 do schemy.
* \[x] Pridat test pad s GND (na pripojenie sondy multimetra/osciloskopu).



### Navrh DPS

Pri navrhu DPS zohladnit nasledujuce pripomienky:

* \[x] Pri ploskach spojenych s GND nastavit uzsie (tensie) mostiky, ktore prepajaju
  plosku s okolitou "vyliatou" plochou, resp. s vrstvou GND. Pri rucnom pajkovani
  sa cez mensie mostiky odvadza menej tepla do susednej velkej medenej plochy
  a suciastky sa lahsie osadzuju.
* \[ ] Popisok konektora dat dalej od samotnej suciastky (aby sa po osadeni nezakryl).
* \[x] Zvacsit diery pre hlavny vypinac (bolo treba zbrusit z pinov).

  * \[x] TODO: diery pre kontakty alebo upevnovacie diery?

* \[ ] Mensie diery pre kolikove konektory (ak budu pouzite, boli volne a dali sa osadit krivo :)
* \[x] Doplnit popisky ku test-padom "TPx" aby sa dali lahsie sparovat so schemou.
* \[ ] Pri vyrobe pouzit ciernu farbu plosaku, nie zelenu. Cierne pozadie zlepsi
  citatelnost 7-seg displejov.
* \[x] Niektore TP presunut ku okrajom DPS, tak aby boli dostupne z oboch stran DPS
  a neboli zakryte suciastkami (napr. displejmi):

  * TP6 (5V)
  * TPxy (3V3, pridat)
  * TPxy (GND, pridat)

* \[x] Antenny konektor moze byt bud SMA alebo U.FL (IPEX)

  * Na DPS cez seba prekryt plosky pre oba typy konektorov
  * Vid. footprint: https://meshtastic.org/assets/images/t-beam-sx1262-3dc0a8d2d866b06b04a457567063e034.webp
  * Bud sa osadi SMA konektor (uhlovy alebo priamy) alebo sa osadi U.FL konektor
    a antena sa zabuduje priamo do skatulky.

* Pri rozmiestnovani suciastok je mozne umiestnit niektore aj pod maly dolny
  displej z TOP strany.

  * Aby bol displej v rovine s RGB displejmi, musi byt osadeny co najdalej od DPS.
  * Tym vznikne pod displejom pohodlny priestor pre SMD rezistory, kondiky, atp...

    * \[ ] TODO: zmerat vzdialenost od DPS po display.



## Mechanicke upravy

* Zachovat rozostup aj velkosti montaznych dier v troch rohoch.

  * Pouziju sa bud na uchytenie do pristrojoveho panela v lietadle alebo na
    uchytenie krabicky.

* Ine montazne diery nebudu potrebne. Bateria sa osadi do drziaka v krabicke.
* Zachovat umiestnenie antenneho konektora GPS na okraji plosaku aby sa mohol
  pouzit aj uhlovy konektor a externa antena.





# Pripomienky k v2024

Releventnost nasledujuce pripomienok ku verzii v2024 a ich aplikovatelnost aj na
verziu v2025 treba posudit jednotlivo. Niektore mozu byt pouzite aspon ako inspiracia
pre finalnu kontrolu DPS pred odoslanim do vyroby.

* Chyba popisok "U8" pre prislusnu suciastku na DPS
* Popisky DP a DM na J3 nie su rovnakym smerom
* Roztec ESP pinheaderov je mala, nesuhlasi s ESP doskou. Zapichnuta kolikova lista
  sa roztiahne ako vejar.
* nejednoznacne ozn. pre D a C (co som tym myslel?)
* kvoli zahnutemu kablu USB-C na ESP doske posunut ESP dosku o 7mm do lava
  a presunut J4 dole na uroven RST tlacitka (na ESP doske)
  a otocit orientaciu J4 smerom hore
* USB konektor pre GNSS dat dalej od ANT1 kvoli pristupu.



# Poznamky/Uvahy

* \[ ] V pripade zabudovania v lietadle a pripojenia ku palubnej sieti bude potrebny
  dodatocny obvod s:

  * menicom napatia z 12/24V na 5V
  * elektronickou poistkou



# Pripomienky k v2026

# pripomienky/navrhy

**26.6.2026**
Telefonicky konzultovane pripomienky: 

- [x] Upgradovat projekt na KiCAD v10?
- [x] Doplnit barometer na I2C linku 
  - https://eu.mouser.com/ProductDetail/Bosch-Sensortec/BMP581?qs=Li%252BoUPsLEntPL9tlFmcgXg%3D%3D
  - umiestnit na top stranu medzi encoder a maly display (do stredu aby bolo dost miesta na teplovzdusne osadenie)
  - CSB pripojit priamo na VDDIO (zvolit I2C)
  - SDO pripojit na GND
  - INT zapojit na TestPad
  - treba zaradit 10Ohm odpor do serie s VDD a VDDIO?
- [x] Na spolocne katody oboch 7seg displejov DS1 (CC1, CC2, CC3, CCL123) a U402 (CC11, CC22. CC33, CC44) doplnit 0Ohm odpor 0603 (mozno sa pouzije na zrovnanie intenzity stredneho a dolneho 7seg displeja, kedze zdielaju jeden driver a spolocne ovladanie intenzity)
- [x] Zvazit otocenie U103 o 90st v proti smere hodinovych ruciciek (bodka bude hore v pravo, Y+ bude smerovat hore)
  - len ak to nebude prilis komplikovane, je mozne kompenzovat smer osi v SW
- [x] Flash U102 zamenit za iny model a ine puzdro, napajanie 2.7-3.3V: 
  - pouzit male puzdro WSON-8 8x6mm (velkostou sa zhoduje s SOIC8 povodne pouzitej W25Q128JV)
    - Winbond W25N02KVZEIR 2Gb (256MB) (Mouser ma na sklade):
      - https://eu.mouser.com/ProductDetail/Winbond/W25N02KVZEIR-TR?qs=YwPsRIUVAOcmpACOC6b95Q%3D%3D
    - Winbond W25N04KVZEIR 4Gb (512MB) (Mouser nema na sklade) 
      - https://eu.mouser.com/ProductDetail/Winbond/W25N04KVZEIR?qs=YwPsRIUVAOdveyibmg1m8Q%3D%3D
  - existuje aj velke SOIC16 puzdro (s dobrou dostupnostou na Mousri) ale to je prilis objemne:
    - https://eu.mouser.com/ProductDetail/Winbond/W25N02KVSFIR?qs=W%2FMpXkg%252BdQ4KUHHgPqoidQ%3D%3D
    - W25N02KVSFIR 2Gb
    - zmesti sa na miesto povodne SOIC8 ak by sa posunul U103 hore?
    - existuje aj mensia kapacita 1Gb: W25N01GVSFIG
- [x] U602 GND pripojit najskor ku R_E605 a az potom ku GND vrstve, vid AppNote strana 4: The STC3100's ground connection (GND) is used as the reference input for the current measurement and must be connected to the ground side of the sense resistor by a dedicated track.
- [x] Ku R_E605 doplnit poznamku, ze jeho hodnota musi splnat: `Rcg <= 0.08/peak_current_A` a `33mOhm => peak +/-2A`.
- [x] U602 pripojit natrvalo ku baterii. Vypinac dat az za U602. 
  - [x] VIN pripojit priamo na bateriu (aj s filtrom).
  - [x] Doplnit SMD jumper, ktorym sa VCC pripoji bud na VIN (pred vypinacom) alebo na BATT (za spinacom)
    - NIE, VCC bude pripojeny stale na bateriu pred vypinacom aby sa zachoval obsah internej RAM
  - Odber obvodu je len 100uA, bude udrzovat stav pocitadiel v internej RAM aj pri vypnutom ESP co zjednodusi sw.
- [x] Nastavit inu I2C adresu pre display driver U4 
  - signaly A0,A1,A2 (ROW2-0) pripojit cez odpory 39kOhm a cez jednu spolocnu diodu ku pinu AD (COM0), vid datasheet
  - jeden z odporov sa osadi tak aby sa zabranilo kolizii adries s CoulombCounterom
- [x] Ku R_E601 pridat TestPad na stranu ku pinu KEY
- GPS U401: 
  - VDD_USB je pouzite nie na napajanie ale na zistenie pritomnosti USB, musi byt urovne 3V3, odber je cca 1mA
  - [x] Pridat jednosmerny level shifter z 5V na 3V3, ktory sa pripoji na VDD_USB cez pull down rezistor 10kOhm + 1uF kondenzator 
    - Alebo pouzit LDO tak ako na vzorovej GNSS7CLICK doske (AP7331)
  - [x] do serie s USB_DP/M pridat rezistor 27 Ω 5% 0.1W (vid vzor GNSS7CLICK)
  - [x] 1uF kondenzator hned vedla VCC pinu
  - skontrolovat layout DPS pre VCC pin a RF_IN pin podla NEO-M9N_Integrationmanual_UBX-19014286.pdf strana 81 https://content.u-blox.com/sites/default/files/NEO-M9N_Integrationmanual_UBX-19014286.pdf
  - [x] EXTINT vytiahnut na TestPad
  - [x] pin Reset vytiahnut na TestPad
- [x] Su niekde v plosaku "slepe prekovy"? T.j. take co nejdu napriec celym plosakom ale len od vonkajsej po niektoru vnutornu vrstvu? 
  - Nemali by sme mat, predrazilo a skomplikovalo by to vyrobu. Na nasom plosaku by sa tomu malo dat vyhnut.
  - NIE, KiCad taketo prepoje by default nerobi
- [x] Prekovy v ploskach urcenych pre rucne osadzovanie sa daju prezit. Ale pri strojovom osadzovani by do nich vtahovalo cin a potom by nozicka nebola pripajkovana (skusenosti z roboty). Idealne ak by prekovy boli mimo padov - aspon pre sucsiastky, ktore maju potencial na strojove osadzovanie (t.j. tie, ktore sa tazko osadzuju rucne).
  - Pokusit sa presunut prekovenia tam kde sa to da. Pri zvysnych prekovoch sa spolahneme, ze sa pri vyrobe PCB (cez JLC PCB) upchaju diery a nebudu stahovat cin.
- [x] Prekovy mimo pajkovacich ploch prekryt nepajivou maskou (farbou). Hlavne tie, ktore su blizko pajkovatelnych ploch (aby do nich nevtahovalo cin a suciastku)
  - KiCad to tak robi by default
- Doplnit menic z externeho zdroja 12-30V na 5V/3A 
  - Externe napajanie bude privedene cez konektor, asi cez Zeleny Konektor - este upresnit !!!
- [x] Ku TP7 doplnit popisok "3V3"
- [x] Ku TP103 doplnit popisok "5V"
- [x] Odstranit popisok "+5V" pri C406
- [x] Doplnit popisky s vyznamom LED: D101 "3V3", D102:"5V", D103:"VUSB"
- [x] Do schemy ku jednotlivym IO z ESP doplnit popisok s vyznamom: (podobne ako tam uz je SCK, MOSI, ...) 
  - IO0: RTS
  - IO2: LED
  - IO4: SEL_FL
  - IO12: TP12
  - IO13: RGB_DATA
  - IO14: PULSE
  - IO15: SW2
  - IO16: GNSS TX
  - IO17: GNSS RX
  - IO25: ENC SW
  - IO26: ENC CLK
  - IO27: ENC DT
  - IO32: SEL_IMU
  - IO35: LIGHT_SENS
- [x] Ku R_E605 doplnit "1%"
- [ ] Ku J601 doplnit typ konektora (PH2.0?)
  - zistit typ podla konektora na baterii
- [x] Doplnit externy PullUp rezistor na IO5 (vyber modu pri bootovani)
- [x] Kozmeticke upravy v scheme: 
  - Strana6: Premiestnit odkaz na USB a BATT na najblizsiu krizovatku cesticiek s bodkou
- [x] Ku kazdemu I2C cipu pridat jeho adresu:
  - `0xEA` BatteryCharger
  - `0x70` CoulombCounter
  - `0x71-7` DisplayDriver, nastavuje sa cez ROW0-2
  - `0x46` Baro
  - `0x44` Lux meter
  - `0x14` Magnetometer
- [x] Vsade na DPS doplnit popisky s oznacenim (nazvom) suciastok
- [x] Doplnit snimac svetla na I2C zbernicu OPT3001 alebo OPT3004DTSR (package SOT-5X3)
  - umiestnit blizko fotorezistora
  - pin ADDR pripojit na GND
- [ ] Overit, ci staci 600mA na 3V3 pre ESP aj GNSS aj ostatne obvody?
  - Na prvom gerete bol rovnaky zdroj 600mA
  - Pribudli nove obvody (baro, flash, imu, magnetometer)
- Vymenit nabijaci cip za USBC-PD, ktory poskytuje nabijanie 1S LiXX clanku


# Dokoncit:

- ???

