- model je zjednoduseny oproti skutecnosti
- zachycujeme jen podstatne veci
- chovani simulace by melo odpovidat chovani systemu
- vyhoda
    - lze pouzit i pro dynamicke sytemy (analyticke reseni lze pouzit jen pro stacionarni systemy)
- pouziti
    - lze simulovat system ktery jeste neexistuje (napr novy kruhovy objezd)
    - da se pouzit pro vycvik (viz simulator letadla)
- model simulace je reprezentovan soustavou matematickych a logickych pravidel
    - vetsina veci je parametrizovatelna
    - rozklad chovani systemu na menzi casti ktere lze dobre popsat + interakce s ostatnimi entitami
- deleni simulaci
    1) dle ucelu
        - analyticke simulace (nepotrebuji interakci s uzivatelem, napr doprava)
        - virtualni prostredi (typicky simulatory)
    2) dle chovani 
        - deterministicke (zadna nahoda) napr problem 3. teles
        - stochasticke (generovani pseudo-nahodnych cisel)
    3) dle casu
        - staticke (cas neni dulezitu, napr vypocet cisla PI)
        - dynamicke (system se vyviji v case)
    4) dle reprezentace casu
        - spojite (stav systemu lze spocitat v kazem okamziku)
        - diskretni (stav systemu lze spocitat pouze v nekterych bodech)
            - ekvidistantni kroky
            - udalostni simulace (kalendar = akce + kdy se maji provest)
    5) dle urovne detailu
        - makroskopicke (jen agregovane hodnoty napr tok na silnici)
        - mesoskopicke (homogenni skupiny entit napr kolony)
        - mikroskopicke (jednotlive entity napr auta)
        - nanoskopicke (detailni chovani entit napr chovani ridicu)
    6) dle typu modelu
        - analyticke (nemuzeme vzorcem pospat cely system ale muzeme popsat jeho casti)
        - numericke (analyticky vzorec prilis slozity viz pocasi)
        - logicke (automaty, sada pravidel)

- hybridni simulace = kombinace vicero typu simulaci; kombinace SW/HW simulace

- vhodne pouziti simulace
    - pri navrhu novych sytemu (musime realnym systemum dostatecne rozumet)
    - pro urceni pozadavku na system (zatez atd - servery, doprava, ...)
    - pri vycviku pro pouzivani jiz existujiciho systemu (leadlo, tank, ... )
    - pri uprace / oprimalizaci systemu

- nevhodne pouziti symulace
    - kdyz je mozne problem vyresit jednoduchou uvahou (viz co se stane kdyz uzavru hlavni tepnu na bory)
    - kdyz existuje snadne analyticke reseni
    - kdyz je realny experiment snazsi (realita je vzdy spravne)
    - kdyz systemu dostatecne nerozumime

- zakladni pojmy
    - entita / agent
        - popsan sadou atributu (rychlost, pozice, ...)
        - kolona, auto, pozadavek, ...
    - stav
        - sada hodnot popisujici system v case
        - `SIR`, `enum { Susceptible, Infected, Recovered }`
    - aktivita
        - cinnost trvajici nejakou dobu
        - napr obsluha pozadavku, let letadlem, ...
    - udalost
        - okamzita zmena stavu entity
        - napr prichod pozadavku

- jak realizovat simulacni experiment?
    - urceni cilu a dostupnych prostredku
    - tvorba modelu
    - overeni modelu!
    - navrh experimentu
    - spusteni experimentu (pokud se jedna o stochastickou simulaci - musime spustit `n` krat => prumer)
    - shromazdeni vysledku (pozor muze jich byt hodne pokud nejsou jiz agregovane!)
    - analyza vysledku (formulace zaveru experimentu)

- citlivostni analyza
    - jaka je citlivost simulace? (zmena vstupu, pozoruju zmenu vystupu)
    - vysoka citlivost nemusi znamenat chybu viz simulace sireni covidu (jeden clovek znovu spusti pandemii)

- nastroje pro tvorbu simulaci
    - obecne programovaci jazyky
        - rucni programovani entit (Java, C++, ...)
        - pouziti existujicich knihoven (JSim, CSim, ...)
    - simulacni programovaci jazyky
        - Modelica, Simula
        - prima podpora simulacnich fci (rychleji implementace, pokud s tim umim)
    - simulacni nastroje
        - netvorime si simulacni model sami
        - napr Simulink (soucast Matlabu), NetLogo
        - "model si muzeme naklikat"

- spojite simulace
    - vetsinou zalozene na diferencialnich rovnicich
    - balisticke krivky, proudeni kapalin, ...
    - byly reseny na analogovych pocitacich
    - v dnesni dobe se dela na digitalnich PC => simulace neni spojita ale ma dostatecne jemny krok
    - obvykle deterministicke modely (zadna pseudo-nahodna cisla)
    - problem N teles
        - jak na sebe pusobi N teles (mumime spocitat pro 2, pro 3 musime pouzit simulaci)
        - spatne se paralelizuje
        - optimalize
            - vliv nekolika vzadlenych bodu lze aproximovat jednim hmotnym bodem (napr pusobeni nejake galaxie na zemi)
            - deleni prostoru mrizkou
                - tam kde je to potreba mohu delit dal a dal
                - kazda podkrychlicka = jedna skupina
            - kdy je vzdalenost dostatecna? => nutno urcit vzorec

- diskretni simulace
    - zalozeny na sade logickych / matematickych pravidel
    - stav simulace definovan v diskretnich bodech v case
    - celularni (bunkove) automaty
        - diskretni jak v case tak i v prostoru
        - prostor rozdelen na bunky o stejne velikosti
        - stav cele simulace se prepocte v kazdem kroku podle danych pravidel
            - => deterministicka simulace
        - novy stav bunky je dan aktualnim stavem bunky + staven budek v jejim okoli
            - => nusim mit pomocne pole pro vypocet noveho stavu
        - nelze odvodit jaky bude `n`-ty nasledujici stav => musim udelat vsechny kroky
        - pomoci jednoduchych pravidel lze modelovat pomerne komplexni prostredi
        - priklady
            - von neumannuv stroj (stroj co sam sebe replikuje)
            - hra zivota
            - dopravni simulace