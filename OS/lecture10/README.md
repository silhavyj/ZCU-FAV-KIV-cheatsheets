- none-real time
    - tradicni os kde se vlakna vykonavaji maximalni rychlosti
    - uzivatele zajima nejkradsi cas dokonceni

- soft-real time
    - vypocet (reponse systemu) nemusi byt hotovat ASPA ale musi byt hotova vcas
    - nedorzeni deadline vede k chybe
    - u soft-real timu muze byt chyba tolerovana

- hard-real time
    - promeskani deadlinu vede k neopravitelne chyba
    - napr nuklearni reaktor, rizeni vozidel, inzulinova pumpa, obrane systemy

- moucha tsetse
    - usi ji slysi, oci ji vidi
    - mozek je embeded system ktery ridi ruku
    - none-real time (traditional system)
        - ruka (vlakno) se pohla prilis rychle, mocha to zmercila a uletela
    - soft-real system
        - ruka se pohla prilis pomalu, moucha uletela
        - nastesti tahle moucha nebyla infikovana (bylo to jen otravne)
    - hard-real time
        - ruka se pohla prilis pomalu, infikovana moucha nas kousla -> nezotavitelna chyba
        - bude alespon protilatka dorucena pred deadlinem?

- komplexnost algoritmu
    - quick vs heap sort
        - oba maji stejnou casovou slozitor O(n*log(n))
        - nejhorsi casova slozitost quicksortu je O(n^2) ale je obvykle rychlejsi nez heapsort
        - heapsort ma nejhorsi casovou slozitor O(n*log(n))
        - actoliv je quicksort rychlejsi, pro RT se hodi vic heapsort - nejhorsi cas je mensi
    - heapsort je vice predikovatelny
        - tim padem je snazsi dokazat stabilitu systemu
    - mohou existovar i dalsi kriteria
        - vyuzita pamet, moznost paralelismu atd.

- Artificial Intelligence (AI)
    - pokud rozumime problemu a vime jak ho vyresit muzeme garantovat maximalni nejhorsi vas jaky to bude trvat - tento pristup je vhodny pro RTS
    - pokud rozumime problemu ale nevime jak ho vyresit muzeme pouzit evolucni algoritmus (EA)
    - Zname maximalni pocet generaci -> zname maximalni nejhori cas vypoctu
    - Pokud nerozumime problemu a tim padem nevime jak ho vyrsit, jsme odkazani pouze na neuronove site
        - nemuzeme vysvetlit na co presne je NN trenovana, nemuzeme nic zajistit, pouze nejhorsi cas vypoctu

- Predpovidani urovne glukozy
    - kriticke embeded zarizeni pro medicinske systemy

- Predikce a bezpecnost
    - real-time systemy (RTS) jsou casto embeded zarizeni
        - omezene zdroje jako je pamet nebo vypocetni vykon
    - RTS ma cenovou funkci pro urceni dopadu promeskaneho deadline - cena kterou musime zaplatit za selhani
    - musime dokazat ze implementace RTS jsou spravna

- Terminologie
    - Task - sekvencne vykonavana cinnost
    - Job - sekvence jednoho nebo vice tasku
    - Resource
        - aktivni - procesor
        - pasivni - mutex, file handler
    - Schedule - plan jak je budou vykonavat jednotlive tasky
        - spravny plan je defakto poradi vykonavani tasku ktere splnuje dane podminky

- Parametery
    - cas prichodu ai - cas kdy i-ty task bude pripraven na vykonavani
    -vypocetni cas ci - cas potrebny pro dokonceni i-teho tasku
    - absolutni deadline di - cas do kdyz by mel byt task dokoncen
    - cas startu, konce a uvolneni (si, fi, ri)
    - perioda pi - interval mezi dvemi aktivacemi i-teho tasku

- Odvozene parametery
    - relativni deadline Di = di - ai
    - response time Ri = fi - ai
    - opozdeni Li = fi - di
        - jak moc je cas dokonceni tasku odpozdeny (muze byt zaporny)
        - kladne cislo porad dovoluje vcasne dokonceni
    - exceeding time Ei = max(0, Li)
    - Slack time Xi = di - ai - Ci = Di - Ci
        - maximalni mozne zpozdeni

- Typy tasku
    - periodicky
        - vykonava se periodicky -> fixni inter-arrival time
        - vsechny instance maji stejny vypocetni cas
    - aperiodicky
        - opakovani aktivovany
        - random inter-arrival time 
    - ojedinely
        - aperiodicke tasky oddeleni minimalni dovou mezi jednotlivymi prichody

- Rate Monotonic Analysis (RMA)
    - prida priority taskum takze:
        - Rate Monotonic Scheduler (RMS) najde naplanovani ktere se da udelat
        - RMA zkontroluje jestli takoveto naplanovani uspeje nebo ne
    - priorita
        - inverzne proporcionalni k periode
        - task s vysokou frekveni ma kratnou periodu
    - Ratio grid <min_period, max_period>
        - pomer mezi sousednimy periodami - 1ms, 2ms, 4ms, ...
        - idealne chceme tolik priorit kolik mame tasky -> ne vzdy mozne

- RMS test naplanovatelnosti
    - Lui a Layland (1973) dokazali existenci uskutecnitelneho naplanovani pro mnozinu periodickych tasku jejihz kolektivni vyuziti CPU je vzdy mensi nez 70%
        - U - vyuziti CPU
        - n - pocet tasku
        - Ci/Pi - vyuzici CPU i-tym taskem
        - `U = sum[i,n](Ci/Pi <= n(sqrt(n,2) - 1))`
    - Pravidelne planovane servery (bottom half) vykonavaji aperiodicke a ojedinele tasky
    - Lui a Layand uvazovali prilis mnoho restrikci
        - mnoho znich by se dalo zanedbat
        - napri uzavovali 0 kernel overhead coz neni realisticke
    - pokud test neprosel, korekni naplanovani stale mohlo existovat ale muselo se to dokazat jinym zpusobem - napr s pouzitim testu dobu odezvy

- Test doby odezvy
    - Spoctemez Rk pro kazdy task
    - `R0 = sum[1,i](Cj)`
    - `Rk+1 = Ci + sum[1,i-1](Ci * ceil(Rk/Pj))`
    - iteruj dokud Rk neprestane rust
    - tasky jsou serazene podle priority
        - secteme casy vsech tasku s vyssi prioritou
            - mensi perioda znamena vyssi priorita
    - pokud Rk < di, potom i-ty task muze byt naplanovany
        - pokud je toto splneno pro vsechny tasky - feasible

- Nejblizsi deadline jako prvni
    - 