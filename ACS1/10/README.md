- predchazeni datovym hazardum dynamickym plnovanim
  - vyhody dynamickeho planovani: HW meni poradi instrukci se zachovanim toku dat a chovanim pri vyjimkach
  - osetruje pripady kdy neni mozno zavislosti urcit v dobe prekladu
    - to dovoluje procesoru tolerovat nepredikovatelna zpozdeni jako napriklad vypadek cache tak ze provadi jine instrukce dokud cache vypadek nezpracuje
  - zaroven umoznuje efektivni beh programu prelozeneho pro jednu pipeline strukturu na jine
  - zjednodusuje prekladac
  - HW spekulativni techniky - prinaseji podstatne vykonove zlepseni, jsouli zalozene na dynamickem planovani

- HW schemate: Instrukcni paralelismus
  - zakladni myslenka: provadet nezavisle instrukce za mistem pozastaveni

  <img src="../img/10/01.png">

  - umoznuje "out-of-order" provadeni a dovoluje "out-of-order" dokonceni (zde napr `SUBD F12, F8, F14`)
    - v dynamicky planovane pipeline strukture vsechny instrukce prochazeji do vkladaciho stupne (issue stage) v puvodnim poradi
  - rozlisujeme kdy se instrukce zacne vykonavat a kdy se dokonci
    - mezi temito dvema casy se instrukce provadi
  - pozn.: dynamicke provadeni zpusobuje hazardy WAR a WAW a osetreni vyjimek je slozitejsi

- Scoreboarding
  
  <img src="../img/10/02.png">

  - faze zpracovani instrukci (instruction execution cycle)
      1) Nacteni (IF = isntruction fetch) - nacteni instrukce s pameti + zvyseni PC
      2) Dekodovani (ID = instruction decode) - dekodovani instrukce; nacteni hodnot a parametru
      3) Provedeni (EX = execute) - provedeni operace nad daty
      4) Pristup do pameti (MEM = memory access) - jen pro load/store instrukce
      5) Zapis vysledku (WB = write back) - zapis vysledku do registru

  - 4 stupne rizeni scoreboardu
    1) dekodovani instrukci & kontrola strukturalnich hazardu (ID1)
        - jeli funkcni jednotka pro zpracovani instrukce volna a zadna jina aktivni instrukce nema stejny cilovy regist (WAW) doda scoreboard instrukci do funkce jednoty a aktualizuje jeji interni datovou strukturu
        - existuje-li strukturni nebo WAW hazard => vlozeni instrukce je pozastaveno a zadne dalsi instrukce nejsou vkladany dokud nejsou hazardy odstraneny
        - algoritmus
          - zajisti vkladani "In-Order"
          - umoznuje provedeni vetsiho poctu operaci behem jednoho cyklu
          - kontroluje zda cilovy registr je jiz rezerovan pro zapis (WAW)
          - kontroluje zda je volni vstup funkci jednotky (mozny strukturni hazard)
    2) cteni operandu: ceka dodud nezmizi datove hazardy, potom precte operandy (ID2) - prvy funkci stupen pipeline
        - zdrjovy operand je k dispozici jestlize zadna drive vlozena instrukce se jej nechystat zapsat ani do nej prave nezapisuje zadna funkci jednotka
        - jakmile jsou zdrojove operandy k dispozici, scoreboard prikaze funkci jednotce provest cteni operandu z registru a spusti provadeni
        - scoreboard vyhodnoti dynamicky v tomto kroku RAW hazardy a instrukce mohou byt poslany do provadeci jednotky bez dodrzeni poradi (out of order)
        - algoritmus
          - cekani na operandy dokud nejsou k dispozici, stav registru vysledku (RAW)
          - ukladani operandu do cache je povoleno
          - Forwarding z jineho stupne WB (write-back faze) je povolen
    3) Execution: operace s operandy (EX)
        - funkci jednotka zacne po obdrzeni operandu provaded operaci
        - kdyz je vysledek hotov, oznami scoreboardu ze operaci dokoncila
    4) Write result: dokonceni provadeci faze (WB)
        - jakmile scoreboard zjisti ze je funkci jednotka dokoncila operaci, scoreboard otestuje vyskyt WAR hazardu
        - neexistuji-li, zapise vysledky jinak pozastavi instrukci
        - algoritmus
          - opozdi zapis dokud vsechna pole `Rj` a `Rk` tohoto registru nejsou oznacena jako "cached" nebo "read"
            - jsou-li operandy v cache, jsou odpovedi (ziskane operandy) "orwardovany" dal
            - v opacnem pripade se zapisem ceka dokud nejsou vsechny operandy precteny
          - forwarduje odpovedi jednotkam cekajicim na tento zapis jejich operandu
  
  - tri casti scoreboardu
    1) stav instrukce
        - indikuje ve kterem ze ctyr kroku zpracovani se instrukce nachazi
    2) stav funkcni jednotky
        - indikuje stav funkcni jednoty (FU); 9 poli pro kazdou funkcni jednotku
          - Busy - indikuje zda je jednotka v cinnosti ci nikoliv
          - Op - operace ktera se ma v jednotce vykonat (napr +/-)
          - Fi - cilovy registr
          - Fj, Fk - cisla zdrojovych registru
          - Qj, Qk - funkcni jednotky generujici zdrojove registry Fj, Fk
          - Rj, Rk - flagy indikujci ze Fj a Fk jsou pripraveny a dosud nebyly precteny (alternativne: byly precteny a ulozeny do cache)
    3) stav registru vysledku
        - indikuje ktera funkci jednotka bude zapisovat kazdy registr, pokud existuje
        - zustava prazdny pokud zadna probihajici instrukce nebude provadet zapis do toho registru
  
- Scoreboard priklad

    <img src="../img/10/03.png">

  - R2 nebyl dosud precten (a umisten do cache) az do cyklu 2

    <img src="../img/10/04.png">

  - druhe LD je blokovane strukturnim hazardem (jen jedna jednotka pro nacitani)

    <img src="../img/10/05.png">

  - dokonceni prvni instrukce (LD) -> funkci jednotka je volna

    <img src="../img/10/06.png">

    - muzeme zacit zpracovavat druhou instrukci LD

    <img src="../img/10/07.png">

    - rovnou muzeme zacit zpracovavat instrukci MULTD ale ta se zablokuje kvuli RAW hazardy z predchozi instrukce kde se zapisuje F2 (cilovy registr)

    <img src="../img/10/08.png">

    - MULTD prosla fazi dekodovani ale je stale blokovana kvuli F2 -> muzeme ale zacit dekodovat dalsi instrukci (SUBD)

    <img src="../img/10/09.png">

    - fetch instrukce SUBD byl dokoncen => muzeme zacit fetchovat instrukci DIVD a zaroven se konecne dokonci WB 2. instrukce LD ktera blokuje MULTD a SUBD

    <img src="../img/10/10.png">

    - instrukce MULTD a SUBD mohou konecne nacist operandy
    - ADDD nemuzeme zacit dekodovat protoze je blokovana funkci jednotka operaci SUBD (v tomto pripade FU = ALU)
    - pozn.: kazda FU muze byt zabrana na jiny pocet instrukcnich cyklu (nasobeni trva dele)
  
  - v tomto duchu se pokracuje dokud nejsou vsechny instrukce dokonceny

- scoreboarding souhrn
  - omezeni scoreboardu u CDC 6600
    - zadny forwarding HW (=> operandy jsou jen v registrech)
    - malu pocet funkcnich jednotek FU (strukturni hazardy)
  - klicova myslenka: nechame provest instrukce lezici za pozastavenou instrukci
    - dekodovani => vlozeni instrukci a cteni operandu
    - umoznuje provaded operaci out-of-order => out-of-order dokonceni
  - soucasna vylepseni
    - vsechny operandy se umistuji do cache jakmile jsou k dispozici
    - forwarding
    - pipelining funkcnich jednotek
    - pracuje nejlepse s velkym mnozstvim registru
    - ukoncovani v poradi
    - preciznejsi vyjimky
    - mikroprogramovani
  - tomasulo metoda
    - rezervacni stanice vs forwarding a caching
    - "docasne" registry jsou vyuzivany jako velke mnozstvi virtualnich registru
