## 01_02

- historie (generace PC, mooruv zakon)
- logicke obvody (digital vs analog, kombinacni vs sekvencni, popis, hradla, bloky, charakteristika)
- booleovska logika, hazardy
- zobrazeni cisel (zakladni zapis cisel, pozicni system)
  - primy kod, inverzni kod, doplnkovy kod, IEEE754
- ALU (popis, vstupy, vystypu, overflow, carry, zero, control, ...)
- HA, FA, FU ze dvou HA
- RCA (zpozdeni, generace, propagace, anihilace)
- paralelni asynchronni sumator
- CLA (odvozeni, vyhody & nevyhody, zpozdeni, serializace CLA, stromova struktura)
- Carry-Skip Adder (propagace vs vypocet, promenne delky bloku)
- Carry-Select Adder (vypocetni slozitost, plocha, velikost bloku, vyhody & nevyhody)
- CIA (Carry-Inc Adder) (vylepseni Carry-Select Adder, vyuziti P,G signalu => vyber vysledku pre XOR)
- odcitacka (dvojkovy doplnek -> +1 realizovano pres Cin = 1)
- seriova scitacka
- citace
  - straight ring counter (manual set 1 pomoci reset)
  - twisted ring counter

## 03

- typy metod nasobeni (sekvencni, paralelni - iteracni aritmeticka pole, specialni kodovani)
- posuvne jednotku (paralel/serial IO, smer, pocet kroku)
  - univerzalni posuvny registr
  - Barrel Shifter
  - Posivna jednotka multiplexerem
  - logaritmicka posuvna jednotka
  - posuvna matice
- parcialni souciny (nasobenec, nasobitel, ...)
- viceoperandove scitani (registr parcialniho souctu, volba typu scitacky, superlinerni vs logaritmicky narust casove slozitosti s `n` a `k`)
- vytvareni parcialniho soucinu
  - zpozdeni nlog(n), posun vlevo vs posun vpravo + soucet (=> sirka scitacky)
- seriova scitacka (princip - blokove schema)
  - akcelerace pouzitim vetsiho poctu scitacek (generujeme vicero paracialnich soucinu naraz -> pouziti vice scitacek seriove)
  - odstraneni serioveho zapojeni (nevyhoda) => vicevstupove scitacky, stromove zapojeni (HW narocne)
- nasobeni zapornych cisel
  - zaporny nasobenec (expanze sign bitu)
  - zaporny nasobitel (odecteni korekcniho clenu (-A))
  - => nevyhoda epanze znamenkoveho bitu (hodne scitani 1)
- boothuv algoritmus - (2,1); (3,2); (4;3) ; s vetsim posunem redukuje pocet parcialnich soucinu
  - lze uplatnit i pro nasobeni s jinym zakladem nez 2 (napr 4) => lepsi je provest prekodovani abychom pouzivali cifry co jsou mocninou 2 (napr {0,1,2,3} -> {-2,-1,0,1,2})
  - odvozeni dle YT videa
- paralelni algoritmy nasobeni
  - redukce pomoci paralelnich dilcich nasobicek (3,2) - FA, (5,5,4)
  - redukcni posloupnost
- Carry-Save Adder = (3,2) redukce
- Wallace nasobicka
- iteracni aritmeticka pole
  - naivni postup
  - cile uprav
  - Baugh-Wooley nasobicka (+ schema s negaci MSB)
  - generovani elementarnich soucinu uvnitr pole (sireni cary, delka kriticke cesty)
    - struktura jedne bunky nasobicky (muze implementovat vicero funkci)
  - piplining
  - pole pro nasobeni s carry-save-adder

## 04

- moznosti implementace delicek (odcitani, newton, deleni konstantou, prevod do log systemu)
- LNS (log2(|X|))
- RNS (+ zaporna cisla, vyhody, nevyhody - deleni obtizne, porovnani je slozite protoze neni pozicni, ucinnost)
- MRS (je pozicni -> jednoduche porovnani; priklad (0,2,2)RMS(5,3))
- ciselny system se zapornym zakladem
  - suda pozice +, licha - (zacina se od 0)
  - rozsahy oboru hodnot v zavislosti na poctu cislic, prenos carry, nevyhoda math. operaci, aplikace zpracovani signalu
- ciselne systemy s ciframi se znamenkem
  - symetricke vs nesymetricke rozlozeni kolem 0
  - algoritmus prevodu {0,1,2,3} -> {-1,0,1,2}
  - algoritmus prevodu [0,3] -> [-2,2] = [-2,1] + [0,1] (dekompozice? "carry free")
- moznosti jak se vyporadat s dlohou prenosovou cestou (detekce ukonceni, CLA, rozsireni poctu bitu vysledku?)
- scitani cisel s redundanci (dekompozice)
  - scitani dvou cisel z [0,18] nam da vysledek [0,36]. My ale chceme vysledek z [0,18] => soupnuti o rad [0,18] -> [0,2] + [0,16]
  - [0,11] -> [0,9] + [0,2] (scitani cisel o zakladu 10 v mnozine cifer [0,11])
- index redundance: `idx = -A+B+1-r`

## 05

- typy kanalu (s pameti - shluky chyb, bez pameti - nezavisle chyby)
- kod (symetricky vs nesymetricky, jednosmerny, blokovy, systematicky vs nesystematicky)
- hammingova vzdalenost
- jednotuche kody
  - opakovaci
  - koktavy
  - parita (licha vs suda - XOR 1); detekce jednoduche chyby bez moznosti opravy; implementace pomoci Moorovo a Mealyho automatu (rozdil mezi nima?)
    - podelna, pricna parita (hamm. vzdalenost = 4 -> detekce az 3 a oprava 1 chyby)
  - checksum (soucet) - nevyhody 0, poradi
  - fletcher checksum (1 byty, modulo 255)
- hamminguv kod
  - 2^r-1 az 2^r-1-r kde r je jedundance
  - generujici matice, kontrolni matice
  - priklad (7,4)
    - hammingova vzdalenost = 3 (detekce dvou oprava max 1 chyby)
    - vypocet paritnich bitu, HW zapojeni
  - rozsireny haminguv kod (pridani parity)
- ECC vs EDC, SRAM vs DRAM, NAND flash layout
- CRC kody

## 06

- ISA (mechanismus SW rika HW co delat, co definuj dobre rozhrani, efektivita HW + abstrakce SW)
  - otazky pri navrhu instrukcni sady
  - pevna vs promenna delka instrukci (vyhody & nevyhody)
  - vyhody velkeho vs maleho poctu registru
  - architektury z pohledu kde jsou ulozene operandy (vyhody + nevyhody)
    - zasobnikova, ACC, memory-memory, register-memory, load-store
    - ortogonalita ISA
- vyhody a neyvhody pouzivani registru
- adresace pameti (word, big endian vs little endian)
- typy adresnich modu
  - direct register, immediate, offset (displacement), index, indirect register, direct memory, indirect memory, auto inc, auto dec, scaled
  - navic vyjimky kdy se adresuje PC
- typy instrukci (aritmeticko-logicke, data transfer, control, system, floating point, string, graphics)
- formaty instrukci (MIPS r, i, j)
- podmineny skok (relativni vuci PC, cykly, pole, ...)
- paralelni systemy
  - grid computing, cluster computing, SMP, multi-core CPU
  - flynnova taxonomie
  - pamet paralelnich systemu (distribuovana vs sdilena - IPC)
    - SMP, NUMA (obrazky, vyhody, nevyhody), + interconnect
- logicky vs fyzicky adresni prostor (L=F, L>F, L<F)
- segmentace (code, data, zasobnik, ...)
- strankovani (algoritmy vyhazovani stranek)
- SRAM vs DRAM (nutnost ECC)
- sbernice (datova, adresni, ridici)
- cache
  - casova vs prostorova lokalita dat
  - propis do hlavni pameti (primi zapis, zapis s mezipameti, zpetny zapis, zpetny zapis pouze jiz propsaneho)
  - velikost bloku, hit rate, isntrukcni vs datova cache
  - info ulozene v cache: data, adresa, dirty bit, info o pouziti polozky
  - algoritmy invalidace cache zaznamu: random, FIFO, LRU, MFU, ..., razeni cachi
  - plne asociativni cache
  - cache s primym mapovanim
  - N-cestna cache
