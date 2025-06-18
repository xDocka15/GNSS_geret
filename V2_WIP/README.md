# Zadanie pre GNSS Geret v2025

Dokument popisuje zmeny/upravy v elektronike gnss-geret z 2024 tak aby vznikla
jednodoskova prenostitelna verzia.  

## Strucny opis zmien

Cielom je dosiahnut vyssiu integraciu, mensie rozmery a odstranit nedostatky 
z predchadzajucej verzie.

Hlavne zmeny:

- [x] Z troch DPS zredukovat len na jednu dosku.
- [X] Povodnu hotovu procesorovu dosku FireBeetle2 nahradit samostatnymi obvodmi 
  priamo na DPS a usetrit tym miesto:
  - [x] Pouzit rovnaky ESP32-WROOM-32E modul
  - [x] USB/UART prevodnik pre programovanie a ladenie ESP32 aj s DTR+RST pre programovanie
  - [x] Menic napatia z baterie na 3V3
  - [ ] Nabijaci obvod pre bateriu (lepsi ako na FireBeetle2)
- [ ] Doplnit flash pre zaznam dat.
- [ ] Lepsia citatelnost displeja s casom.

Vacsina suciastok zostava zachovana, zhodna s verziou v2024. 



## Nove suciastky

### ESP32

Zvolit modul:
- ESP32-WROVER-E-N16R8 - trosku vacsi rozmer, ale ma vacsiu pamat, vyvody len z dvoch stran.
- ESP32-WROOM-32E-N16R2 - mensi rozmer, mensia pamat, vyvody z troch stran.
- [ ]

Umiestnit antenou ku okraju DPS. Pod vstavanou antenou je potrebne odstranit
oblast zeme v jednotlivych vrstvach DPS tak aby nedoslo ku zatieneniu anteny (napr.
vrstvou GND, vid. keepout zone v datasheete). Idealne vyfrezovat z plosaku.

Doska WROVER sa pripajkuje priamo na hlavny DPS.

Nerozpustat obsah WROOM modulu priamo na hlavny DPS, odovodnenie:
- nepredpokladam, ze by sa podarilo dosiahnut vasiu mieru integracie ako samotnemu 
  vyrobcovi Espressif - t.j. neusetri sa miesto na DPS
- rucne osadzovanie by bolo skoro nemozne
- vyzadovalo by to ozivit zapojenie aj samotneho procesora
- neskor by ake kolvek chyby mohli mat pricinu bud v osadeni alebo navrhu zapojenia 
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

 - [ ] TODO: vyspecifikovat, na ktore piny procesora pripojit. Ktore z dvoch SPI rozhrani je volne?


### Nabijaci resp. power-management modul

Na napajanie bude pouzity jeden clanok Li-Ion baterie (v letectve sa nepouzivaju Li-Pol). 
Mal by obsahovat:
- [ ] moznost vycitat cez komunikaciu (idealne I2C) stav nabijania (miera vybita)
- [ ] coulomb counter na meranie kapacity bateria
- [ ] meranie napatia baterie

Zapracovat modul na sledovanie kapacity LiXy batetie MAX17043.
https://techfun.sk/produkt/max17043-i2c-modul-pre-sledovanie-stavu-baterie-1s/

https://www.sparkfun.com/mikroe-charger-2-click.html
Coulomb counter TC3100 + nabijaci obvod IP5306-I2C.
https://electronics.stackexchange.com/questions/17463/help-on-stc3100-calculations
https://github.com/danirebollo/ArduinoSTC3100

- [ ] TODO: upresnit.


### Tlacidlo
Pridat nove mikro-tlacidlo niekam na pravu prednu stranu. 

- [ ] TODO: Vyspecifikovat tlacidlo, umiestnenie a pripojenie. 


### Indikacna LED

Pridat SMD modru LED na stranu DPS s displejmi. Pouzije sa na zakladnu indikaciu
a ladenie SW. Spolu s LED na rovnaky ovladaci pin procesora pripojit aj test-pad.
Nakolko su vsetky ostatne LED pripojene len cez komunikacne rozhranie, tato bude
urcite fungovat vzdy.
Zapojit na GPIOxy voci GND.

- [ ] TODO: Urcit, na ktory GPIO pin. 


### Senzor osvetlenia

Doplnit senzor osvetlenia a automaticky upravovat jas.
Umiestnit ho do vnutorneho rohu kde sa stretava maly dolny displej a stredny velky displej
na pravej strane (na opacnej ako je encoder, aby sa rukou nezakryl).
V kryte sa na danom mieste urobi vybratie, cez ktore (a cez foliu) bude prenikat okolite svetlo
az ku DPS.

- [x] TODO: Pouzit jednoduchy fototranzistor? Vyspecifikovat.


## Upravy

Vo verzii v2024 boli na DPS/scheme identifikovane nesledujuce body na zlepsenie: 

- [x] Doplnit USB iface pre gnss (3v3 urovne, pripojit vdd_usb)
  Zvazit umiestnenie plnohodnotneho USB micro-B konektora otoceneho ku okraju DPS 
  kvoli servisnemu pristupu do GNSS modulu. Pouzit uhlovy konektor o 90st zarovnany 
  s okrajom DPS. Pristupny bude len po vybrati DPS z krabicky.
- [ ] Pridat odrusovaci kondik na tlacidlo tocitka.
  - TODO: Vyspecifikovat.
- [x] Na vstup A a B z encodra doplnit pull-up rezistory 10k na 3V3.
- [x] Do footprintu encodera pridat aj diery pre upevnovacie otvory (pajkovatelne, spojene s GND)
- [x] Pouzit iny typ stredneho 4-ciferneho displeja:
  - Na v2024 bol pouzity typ so spolocnou anodou (KW4-804AGB).
    - Radic HT16K33 je koncipovany tak, ze:
    - cez samostatne anody dokaze do LED pustit 20mA
    - ces spolocnu katodu dokaze preniest 8x 20mA    
    - kvoli tomu nebolo mozne vybudit LED na maximalny jas co sa prejavilo na zlej 
      citatelnosti za slnecneho svetla.
  - [x] Vo v2025 pouzit model so spolocnou katodou (KW4-804CGB).   
    - Tomu sa musi prisposobit aj zapojenie jednotlivych segmentov (po vzore maleho displeja).
    - [x] TODO: vyspecifikovat zapojenie displeja.    
- [ ] Pouzit USB-C konektor, zapojenie rovnake ako na FireBeetle2.
- [ ] Ku GNSS modulu pripojit aj zalohovaciu bateriu pre RTC.


### Chybajuce veci

Vo verzii v2024 chybali nasledujuce veci, ktore su aplikovatelne aj na v2025:

* Na 5V vetvu pridat velky stabilizacny kondik 470u/16V.
* Doplnit oznacenie D2 do schemy.
* Pridat test pad s GND (na pripojenie sondy multimetra/osciloskopu).



### Navrh DPS

Pri navrhu DPS zohladnit nasledujuce pripomienky:

- [x] Pri ploskach spojenych s GND nastavit uzsie (tensie) mostiky, ktore prepajaju 
  plosku s okolitou "vyliatou" plochou, resp. s vrstvou GND. Pri rucnom pajkovani 
  sa cez mensie mostiky odvadza menej tepla do susednej velkej medenej plochy 
  a suciastky sa lahsie osadzuju.
- [ ] Popisok konektora dat dalej od samotnej suciastky (aby sa po osadeni nezakryl).
- [x] Zvacsit diery pre hlavny vypinac (bolo treba zbrusit z pinov).
  - [x] TODO: diery pre kontakty alebo upevnovacie diery?
- [ ] Mensie diery pre kolikove konektory (ak budu pouzite, boli volne a dali sa osadit krivo :)
- [ ] Doplnit popisky ku test-padom "TPx" aby sa dali lahsie sparovat so schemou.
- [ ] Pri vyrobe pouzit ciernu farbu plosaku, nie zelenu. Cierne pozadie zlepsi 
  citatelnost 7-seg displejov.
- [ ] Niektore TP presunut ku okrajom DPS, tak aby boli dostupne z oboch stran DPS 
  a neboli zakryte suciastkami (napr. displejmi): 
  * TP6 (5V)
  * TPxy (3V3, pridat)
  * TPxy (GND, pridat)
- [x] Antenny konektor moze byt bud SMA alebo U.FL (IPEX)
  * Na DPS cez seba prekryt plosky pre oba typy konektorov
  * Vid. footprint: https://meshtastic.org/assets/images/t-beam-sx1262-3dc0a8d2d866b06b04a457567063e034.webp
  * Bud sa osadi SMA konektor (uhlovy alebo priamy) alebo sa osadi U.FL konektor
    a antena sa zabuduje priamo do skatulky.    
* Pri rozmiestnovani suciastok je mozne umiestnit niektore aj pod maly dolny 
  displej z TOP strany.
  * Aby bol displej v rovine s RGB displejmi, musi byt osadeny co najdalej od DPS.
  * Tym vznikne pod displejom pohodlny priestor pre SMD rezistory, kondiky, atp...
    - [ ] TODO: zmerat vzdialenost od DPS po display.     



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

- [ ] V pripade zabudovania v lietadle a pripojenia ku palubnej sieti bude potrebny
  dodatocny obvod s:
  * menicom napatia z 12/24V na 5V
  * elektronickou poistkou  
