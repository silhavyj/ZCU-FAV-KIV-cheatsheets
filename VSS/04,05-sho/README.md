- prvni prakticka aplikace markovskych modelu
- modelovani systemu ktere obsluhuji vetsi mnozstvi pozadavku
    - rizeni dopravy
    - telekomunikace
    - navrh vyrobnich linek
    - nemocnice, urady, ...
- pozadvky prichazeji do systemu, nejakou doby tam stravi a pak ho zase opousti
- aplikace ppsti a statistiky => vysledky plati pouze pri dlouhodobym pozorovani systemu (dlouhodoby pocet pozadavku)
- jedna se o odhad!
- zakladni koncept
    - kanal obsluhy = mame prvek v systemu ktery realizuje sluzbu (napr server)
    - klienti posilaji pozadavky 
    - pozadavky jsou bud hned oblouzeny nebo cekaji ve frontach

    <img src="../img/04-sho/01.png">

- pokud mame model jednoduchy -> lze nalezt vzorecky (napr kolik bude prumerna delka fronty pozadavku)
    - pokud je model prilis slozity -> musime pouzit simulaci

- zakladni problemy
    - 1) prilis mnoho pozadavku na system
            - lide mohou napr z fronty odejit (typicky fronty lidi lze modelovat jako nekonecny fronty viz vidani noveho iPhonu)
            - nebo muze dochazet k zahazovani pozadavku (packety v routeru)
            - => potrebujeme navisit kapacitu (popularni cloudove resni -> lze dimenzovat sluzbu dle zateze)
    - 2) malo pozadavku na system
            - system vetsinu casu nic nedela (napr zapis predmetu)
            - vyuziti nepoziteho vykonu napr SETI@HOME
                - nevyhoda ze defakto nevime co komu pocitame
            - => mohu zdroje vyuzit nejak jinak nebo je treba zrusit?

- elementarni SHO 

    <img src="../img/04-sho/02.png">

    - elementarni system hromadne obsluhy ma celkem 5 polozek
        - vstupni proud (vnejsi svet co po nas neco chce)
        - fronta
        - pocet kanalu obsluhy co sdili jednu frontu (vice front = vice elementarnich SHO)
        - vystupni proud
    - na zaklade popisu techto polozek muzeme odhadnout jak se bude chovat vystupni proud

    - abychom toto mohli analyzovat analyticky musime predpokladat STACIONARNI REZIM CINNOSTI
        - charakteristiky se nemeni v case
        - jinak musime pouzit simulace
        - napriklad lide na prepazce mohou byt na konci dne unaveny -> zmena parametru rychlosti obsluhy

    - modeluji az ustaleny stav
    
    - zdroje pozadavku
        - modelovani vnejsiho sveta
        - v nejakych situacich mame nejakeho producenta (lepsi situace -> muzeme ho analyzovat = vim kolik cca pozadavku bude chodit)
        - v horsim pripade nemame co analyzovat a musime jen odmerit jak se vnejsi svet chova (kolik chodi pozadavku atd)
        - omezeny zdroj
            - predem znama mnozina pozadavku
            - napr procesy v KS (vim kolik vlaken mam v programu) nebo pool vlaken v DB serveru
        - neomezene zdroje
            - napriklad webovy server
        
    - jmenne konvence

        <img src="../img/04-sho/03.png">

    - vstupni proud
        - da se na nej koukat jako na posloupnost casu mezi pozadavkama
        - doby lze brat jako nahodnou velicinu => muzeme hledat rozdeleni ktere je charakterizuje
        - nejcastejsi popis je distribucni fci nebo hustotou ppsti

            <img src="../img/04-sho/04.png">
        
            - poissonovsky - exponencialni rozdeleni pro doby mezi prichody pozadavku
            - gaussovsky - neni typicke pro lidi - nebudo chodit kazdou minutu (typicke napr pro nejaky PC system)
            - rovnomerny - BEZ NAHODY (stejna doba mezi pozadavky)
        
        - abychom mohli pouzit markovske modely musi byt vstupni proud
            - 1) homogenni = chovani v case se nemeni
                    - ne vzdycky realni viz zatez Netflixu pres den (meni se od hodiny) => rozdelit interval na "konstantni" podintervaly => modelujeme system vicekrat

                    <img src="../img/04-sho/05.png">

            - 2) ordinalni = nikdy neprijde vice nez 1 pozadavek na jednou
                    - jeden pozadavek bude drive nez druhy
                    - v realu muze byt predrazeno neco co zajisti usporadani 
            - pokud tyto podminky nejsou splneny => pouzit simulaci
        
        - vstupni proud lze popsat parametry odpovidajiciho rozdeleni
            - napr. stredni doba mezi prichody `Ta = 1 / λ`
            - `λ` = frekvence prichodu pozadavku
        
        - pokud nemame na vstupu poissonovsky proud
            - musime pouzit koeficient variace `Ca = σ / Ta`
            - = pomer mezi smerodatnou odchylkou a stredni dobou mezi prichody pozadavku
            - "jak moc je proud nahodny"
                - pro exp nebo poisson je `Ca = 1`
                - prlne deterministicky system `Ca = 0`
                - pokud `Ca > 1` => objevuji se shluky pozadavku (nic horsiho mit nemuzeme viz menza)

                <img src="../img/04-sho/06.png">

    - fronta pozadavku
        - maximalni delka (omezena / nekonecna)
        - prioritni / FIFO?
            - prioritni fronta typicky v OS nebo akutni urazy v nemocnici
        - co se deje s pozadavky?
            - opousti frontu (co to znamena? zahozeni packetu vs umrti)
        - analyticke vypocty jsou snadne pro fronty FIFO, bez omezeni, bez priorit
            - jinak => pouzit simulaci
        - charakteristiky fronty
            - `w` - aktualni (okamzity) pocet pozadavku ve fronte (nah. velicina)
            - `E(w) = Lw` - stredni pocet pozadavku ve fronte
            - `t(w)` - doba cekani jednoho konkretniho pozadavku ve fronte (nah. velicina)
            - `E(tw) = Tw` - stredni doba cekani pozadavku ve fronte

    - kanal obsluhy
        - elementarni SHO muze mit `n` kanalu obsluhy (napr pocet jader CPU)
        - potrebujeme charakterizovat dobu obsluhy jedhono pozadavku = nahodna velicina
            - idealne chceme aby se v case nemenila
            - v realu napr obsluhy na prepazkach jsou ke konci smeny unaveny -> jde jim to pomalu nez na zacatku
            - zase predpokladame exp rozdeleni 
            - sterdni doba obsluhy pozadavku `Ts = 1 / μ`
            - rika nam dve veci
                - jak rychle dokazeme odbavovat pozadavky
                - dava nam informaci o tom ze vstupni pozadavky jsou stejny
                    - ze nejsou jinak narocny (viz triska vs amputace v nemocnici)
        - koeficient variace ("jak moc jsou doby obsluhy nahodne")
            - `Cs = σ / Ts`
        - v realnem svete je doba obsluhy spise gaussovo rozdeleni (trva +- stejnou dobu)

    - celkove charakteristiky SHO
    
        - `ρ` zatizeni systemu (`m` = pocet kanalu obsluhy)
        
            - = PPST OBSAZENOSTI KANALU

            <img src="../img/04-sho/07.png">

            - `ρ = 0` system nic nedela
            - `ρ = 1` system je plne vytizeny
            - `ρ >= 1` system nestiha, neni ve stacionarnim rezimu
                - fronta neustale narusta

        - `Lq` = stredni pocet pozadavku v systemu SHO
        - `Tq` = stredni doba odezvy (soucet stredni doby cekan ve fronte + stredni doba obsluhy)
            - pozadavek musi projit jak frontou tak obsluhou

    - vystupni proud
        - sizne zavisly na rezimu prace kanalu
        - pokud je system ve stationarnim stavu (`ρ < 1`)
            - stejna perioda a frekvence jako vstupni proud
            - jinak by se pozadavky hromadily (zakon zachovani)
            - pokud je `ρ` blizke `0` -> vstup = vystup (pozadavky system jen "protecou")
                - pozadavek odejde driv nez prijde dalsi pozadavek
                - casy prichodu pozadavku jsou jen posunute o cas obsluhy
                    - ale charakteristiky proudu jako takoveho jsou zachovany
        - nestacionarni rezim (`ρ >= 1`)
            - vsechny kanaly pracuji
            - frekvence vystupu musi byt `m * μ`
            - rozdeleni stejne jako rozdeleni kanalu obsluhy

        - jaktoze pri `ρ < 1` vznika fronta kdyz system stiha?
            - zpozdovani je dano nahodou (cim vetsi nahota tim je vetsi ppst mi vznikne fronta)
            - je to dano tim jak neni system synchronizovany (nejsou stejne doby mezi prichody pozadavku)
            - pokud system stiha a je synchronizovany => fronty nevznikaji


- zakladni vztahy strednich hodnot (Littlovy vzorce)

    - stacionarni rezim, FIFO, zadna preemce

        <img src="../img/04-sho/09.png">

        - doba obsluhy ze je `1 / μ` neni az tak prekvapive
        - `m * (λ / μ)` je stredni pocet prvku kanalu obsluhy (pocet kanalu `m` krat vytizeni)

        <img src="../img/04-sho/08.png">

    - provazuji spolu dobu a pocet (vyazeji ze zakonu zachovani)
    - neni zavisle na ppstnim rozdelenim vtupniho proudu

        <img src="../img/04-sho/10.png">

    - za dobu `Tq` mi prijde `λ` pozadavku, ale za za dobu `Tq` mi take musi odejit `λ` pozadavku jinak by se v systemu hromadily (SHO by nebyl stacionarni)

- zakladni uloha teorie hromadnych systemu
    - pokud zname chrakteristiku vstupniho proudu `Fa(t)` a kanalu obshuhy `Fs(t)` chceme urcit `Lq, Tq, Lw, Tw`

- Kendallova klasifikace

    <img src="../img/04-sho/11.png">

    <img src="../img/04-sho/12.png">

    - podle typu systemu muzeme v tabulkach dohledat prislusne vzorecky

- nejjednodussi priklad M/M/1
    - poissonovsky vstupni proud `λ`
    - stredni pocet obslouzenych pozadavku za jednotku casu `μ` (ZA PREDPOKLADU ZE `ρ = 1`)
        - pokud je `ρ < 1` -> skutecna frekvence bude nizsi tim jak system po nejakou dobu nic nedela
    - `ρ = λ / μ` (stacinoarni rezim pokud `λ < μ`)
    - lze modelovat jako markovsky proces
        - NUTNO OVERIT ZE JE PROCES STACIONARNI!!

    <img src="../img/04-sho/13.png">

    - jak urcim ppst ze se nachazim ve stavu `pk`?
    - neni zadny absorpcni stav => muzu system popsat rovnicemi (na leve strane je 0)

    - odvodim rovnice pro stav 0 a stav 1 ktere muzu nasledne secist

    <img src="../img/04-sho/14.png">

    - `ρ = λ / μ` to vim to je zadane
    - `p0` neznam...ale `p0` je ppst ze jsme ve stavu 0 (v systemu nejsou zadne pozadavky; system neni zatizeny) => ve vsech ostatnich stavech je system nejak zatizeny
        - => `ρ = 1 - p0` => `p0 = 1 - ρ`
    - jaky je ale stredni pocet pozadavsku v systemu `Lq`? (pro `ρ < 1` jinak cely ten model "poroste" doprava)
        - soucet pres geometrickou radu: `Lq = ρ / (1 - ρ)`
        - `Tq`, `Lw`, `Tw` vypoctu s pouzitim Littlovo vzorcu

- priklad M/M/m

    <img src="../img/04-sho/15.png">

    <img src="../img/04-sho/16.png">

    - vzdycky pocet pozadavku muze obslouzit `x` kanalu (`x <= m`)
    - pomerne jednoduche urcit vzorecky pro konkretni M/M/m system (napr M/M/5) ale je tezke je urcit obecne pro `m` kanalu
    - pokud je ale `m = 1` nebo `m = 2`, existuje PRIBLIZNY odhad:

        <img src="../img/04-sho/17.png">

- priklad M/M/m s omezenou delkou fronty
    - neomezenost fronty => markovsky model je nekonecny
    - konecne fronty -> nektere pozadavky jsou zahozeny
    - napr M/M/2/2/FIFO

        <img src="../img/04-sho/18.png">

    - `p4 * λ` je intenzita zahazovani pozadavku
    - system se nemuze pretizit (kdy bude `λ > μ` tak cast pozadavku zahodim => tim se snizi tok pozadavku na system)
    - stredni pocet pozadavku v systemu `Lq = ` pocet pozodavku v danem stavu * prislusna ppst

    <img src="../img/04-sho/19.png">

    - REALNE ZATIZENI SYSTEMU: `ρ = p1 / 2 + p2 + p3 + p4`
        - `p1 / 2` protoze je system vytizeny na pul (jedna obsluha pracuje a druha ne)
    - `Lw = p3 + 1 * p4` protoze ve stavu `p3` je ve fronte jeden pozadavek a ve stavu `p4` dva
    - REALNA FREKCENCE VSTUPNICH POZADAVKU
        - neni to `λ` (protoze je zahazuji)
        - ale `λ(1 - p4)`, ve stavu `p4` pozadavky zahazuji

- priklad M/G/1
    - v ralite se doby obluh chovaji jinak nez exponencialne (napr gaussovsky)
        - napriklad smerovac v pocitacove siti
    - musime znat 
        - `λ` frekvence prichodu pozadavku
        - `Fs(t)` nebo `fs(t)` popisujici doby obsluhy a prislusne parametry 
    - lze urcit `ρ = λ * Ts` kde `Ts` je stredni doba obsluhy pozadavku
    - pro dalsi musime  vypocty urcit koeficient variace `Cs`
        - rika jak daleko jsem od exponencialniho rozdeleni
        - pro exponencialni rozdeleni (markovsky system) `Cs = 1`

        <img src="../img/04-sho/20.png">

        - muzu pouzit markovsky `Lw` a prenasobit ho koeficientem `(1 + Cs^2) / 2` (jak moc jsme "daleko" od exponencialniho rozdeleni)

- priklad GI/G/1

    <img src="../img/04-sho/21.png"> 

    - pro D/D/1 je `Lw = 0` protoze `ρ < 1` a prichody jsou deterministicky
        - stiham obsluhovat a neni zadna nahoda co by zapricina vznik fronty

- site SHO (vice propojenych elementarnich SHO)
    
    <img src="../img/04-sho/22.png"> 

    - obvyjle blizssi realize (pozadavek prochazi skrz nekolik systemu)
        - LB -> aplikacni server -> DB
    - orientovany graf
        - uzly = elementarni SHO systemy
        - hrany = propojeni mezi systemy
    - pokud nmam zpetne hrany lze resit "trivialne"
        - jednotlive SHO muzeme resit oddelene

    <img src="../img/04-sho/23.png"> 

    - pokud mame zpetne vazby musime system resit jako celek
        - zakladem je urcit charakteristiky vnitrnich toku

    <img src="../img/04-sho/24.png"> 

    - system lze popsat jako orientovany graf
        - hrany jsou vahy prechodu mezi jednotlivymi systemy

            <img src="../img/04-sho/25.png"> 
    
    - site mohou byt
        - uzavrene existuje cesta z kazdeho uzlu do kazdeho (nema absorpcni stavy)
        - otevrena
            - prevedeni na uzavreny system pridanim uzlu pro okolni prostredi
    
    - dulezite je jetli mam omezeny pocet pozadavku nebo nekonecny

- obecne vlastnosti siti SHO
    - musi platit zakon zachovani
    - musi platit ze soucet vystupnich ppst z kazdeho uzlu = 1
    - spojeni dvou toku ktere maji exp rozdeleni da zase exp rozdeleni
    - "jacksonuv zakon"
        - pokud jsou toky exponencialni a sit je stacionarni (neni pretizena) pak vystupni toky budou take exponencialni
    - pozadavky v SHO (abychom mohli resit analyticky)
        - pozadavky (naroky na obsluhu) se nemeni v case
        - napr ppst vyplneni spatne formulare na poste se zmensuje s poctem pokusu
        - problem je kdyz mame treba ruzne tridy pozadavku

- zakladni charakteristiky site SHO
    - zatizeni
    - prutok (kolik pozadavku projde za jednotku casu)
    - stredni delka front
    - stredni doba odezvy (za jak dlouho pozadavek projde siti)
        - kriticke napr v safety-critical systemech

- otevrene site front
    - muze vstoupit libovolne mnozstvi pozadavku
    - system neni zahlceny (`ρ < 1` pro kazdy elementarni SHO)
    - matematicky model nehodi se k modelovani extremnich stavu (lepsi je pouzit simulaci)

- Littleuv zakon

    <img src="../img/04-sho/26.png">

- rovnice toku

    <img src="../img/04-sho/27.png">

    - plati jen za predpokladu ze je system stacionarni
        - pro kazdy system musime znat
            - pocet kanalu
            - stredni dobu oblsuhy kanalu

                <img src="../img/04-sho/28.png">
    
            - pote co tohle plati muzu o systemu neco rict (analyzovat ho)

- priklad financni urad
    
    <img src="../img/04-sho/29.png">

    <img src="../img/04-sho/30.png"> 

    <img src="../img/04-sho/31.png"> 

- uzavrene site front SHO
    - pevna konecna mnozina prvku (pozadavku) ktere v systemu koluji
        - => fronty nemohou rust do nekonecna
    - otevrene site front = "mechanicka zalezitost dosazeni do vzorecku"
    - uzavrene site front musime pro kazdy pripad postavit vlastni model (analyza / vypocet se lisi s kazdym prikladem)

    - jednoduchy priklad

        <img src="../img/04-sho/32.png"> 

        <img src="../img/04-sho/33.png"> 

        - stredni delka fronty je soucet pocet prvku v jednotlivych frontach krat ppst daneho stavu
            - ve stavu 0 je ve fronte 1 jeden prvek
            - ve stavu 2 je ve fronte 2 jeden prvek
        
        - pruchodnost = s jakou intenzitou pozadavek dokonci cyklus (projde obema obsluhama)
            - problem je ze v odpovidajicim markovskem modelu neni zadna jedna hrana ktera by odpovidala hrane `x` (dokonceni cyklu)
                - => vysledna pruchodnost je soucet frekvenci pruchodnosti po obou hranach
        - => tato analyza plati jen pro tento konkretni pripad

    - priklad interaktivni system

        <img src="../img/04-sho/34.png"> 

        - server + n terminalu
        - jednovlaknove zpracovani `μ`
        - uzivatele se pripojuji z intenzitou `λ`

        <img src="../img/04-sho/35.png"> 

        - zajima nas pruchodnost systemem (hrana zpetny vazby)
            - hranou muzu projit jen v pripade ze budu neco obsluhovat
            - => `x = (1 / Ts) * (1 - p0)`
            - ve stavu `p0` nic neobsluhuju (doplnek je ze neco obsluhuju)

    - priklad OS s vice procesy

        <img src="../img/04-sho/36.png"> 

        <img src="../img/04-sho/37.png"> 

        - stav 0 je ze vsechny procesy cekaji na pamet
        - v markovskem modelu by take mohly byt u stavu 1, 2, 3 smycky ktere naznacuji prechod pres hranu `p` do toho sameho stavu
        - pruchodnost systemem 
            - `x = μ * p * (p1 + p2 + p3)`
            - k ukonceni procesu muze dojit pouze v pripade ze jdu po hrane `p` (intenzita prechodu -> `μ * p`) a tato hrana se vystytuje (i kdyz to v tom obrazku neni) ve stavech `p1`, `p2` a `p3`
        - stredni doba obsluhy `Tq = n / x = 3 / x`

- Neopoissonovske site front (priblizne reseni)

    <img src="../img/04-sho/38.png"> 

    - `Cout^2` rika to jak rozdelit koeficient variace v zavislosti na zatizeni a koeficientech variace vstupnihou proudu a kanalu obsluhy
    - nevyhoda je ze nemuzeme snadno scitat toky (tak jako kdyz mame exp rozdeleni)
    - muze byt vhodne misto analytickeho reseni sahnout po simulaci