- prehled pipeliningu
  - CPI (= clocks/cycles per instruction)
  - CPI pipeline = CPI idelani pipeline + vliv strukturalnich hazardu + vliv datovych hazardu + vliv ridicich hazardu
  - CPI idelani pipeline
    - mereny maximalni vykon, ktereho je dana implementace v idelanim pripade schopna
  - strukturalni hazardy: omezeni HW (HW neni schopen soucasneho vykonavani dane kombinace instrukci)
  - datove hazard: instrukce zavisi na vysledku predchozi instrukce ktera je stale v pipeline
  - ridici hazardy: zpusobene zpozdenim mezi nactenim instrukce a rozhodnutim o zmene instrukcniho toku (vetveni & skoky)

- ILP = instruction level paralelism
  - prekryvani instrukci za ucelem zvyseni vykonu
  - dva pristupy
    1) zavisi na HW a vyuziva paralelismus dynamicky
    2) spociva v SW technologiich ktere vyhledavani moznosti paralelismu staticky v dobe prekladu
  - ZB (= zakladni blok programu)
    - je relativne maly
    - = sekvence prikazu bez vetveni (krome vetveni na vstupu a na vystupu)
    - stredni dynamicka frekvence vetveni je 15% az 25%
      - => mezi dvema podminenymi skoky se vykona 4 az 7 instrukci
      - plus instrukce v ZB se zdaji byt zavisle jedna na druhe
    - => pro vyznamne zvyseni vykonu musime vyuzit ILP pres vice zakladnich bloku
    - nejjednoduseji: paralelismus na urovni smycek pro vyuziti paralelismu mezi iteracemi

        <img src="../img/09/01.png">

- paralelismus urovne smycek
  - "rozbalovani" smycek
    1) dynamicky predikci vetveni
    2) staticky - rozbaleni smyccky prekladacem
  - jiny pripad jsou vektory (viz dale)
  - urceni zavislosti instrukci je kriticke pro paralelismus smycek
  - 2 instrukce jsou but
    - paralelni (mohu je provest soucastne bez toho aby se navzajem ovlivnili)
    - zavisle (musi se vykonat v danem poradi)

- zavislosti dat a hazardy
  - instrukce `I` je datove zavisla (prava zavislost) na instrukci `J`

  <img src="../img/09/02.png">

  - instrukce se nemohou vykonavat soucasni ani se uplne prekryt
  - jestlize datova zavislost zpusobila hazard v pipeline, nazva se Read After Write (RAW) hazard
  - dulezitost datovych zavislosti:
    - indikuji moznost vyskytu hazardu
    - urcuji poradi vykonavani instrukci
    - udavaji horni hranici mozneho paralelismu
  - cil pro HW/SW: vyuzit paralelismus pri zachovani poradi instrukci jen tam kde to prinese zisk

- zavislost jmen #1: Anti-dependence
  - pokud 2 instrukce pouzivaji stejny register (nebo misto v pameti) ale neni mezi nimi zadny tok dat

  <img src="../img/09/03.png">

  - zpusobi-li anti-dependence hazard v pipeline, nazyva se Write After Read (WAR) hazard
  - reseni: pozuziju jiny (volny) registr (`J: add r5, ...`) resp prohodim poradi

- zavislost jmen #2: vystupni zavislost
  - instrukce `J` zapise oprand predtim nez zapis provede instrukce `I`

  <img src="../img/09/04.png">

  - prameni z opetovneho pouziti stejneho jmena
  - zpusobi-li vystupni zavilost hazard v pipeline, nazyva se pak Write After Write (WAW)
  - resenim aby se instrukce mohli provest paralelne: zmena jmena (pouziteho registru)
    - reseni na urovni prekladace nebo HW

- zavislosti rizeni vypoctu
  - kazda instrukce je zavisla (control dependent) na urcitem souboru instrukci vetveni (napr `if` statemant)
  - obecne se tyto zavislosti musi zachovat aby byla zajistena posloupnost programu

  <img src="../img/09/05.png">

  - `S1` je control-dependant na `p1`
  - `S2` je control-dependant na `p2` (ale nikoliv na `p1`)

- ignorovani ridicich zavislosti
  - neni je nutne zachovat:
    - chceme-li provest instrukce ktere by nemely byt provedeny -> muzeme to dopustit pokud korektnost programu zustane zachovana (instrukce nemaji vedlejsi efekt)
  - naproti tomu dve vlastnosti jsou z hlediska korektniho provedeni programu kriticke
    - chovani vyjimek
    - tok dat
