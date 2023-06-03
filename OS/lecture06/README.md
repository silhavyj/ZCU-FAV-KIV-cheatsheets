- .h - header file (popisuje co object file a staticka knihovna obsahuje)
- .o - object file
- .lib, .a - staticka knihovna
- .dll, .so - dynamicka knihovna

- IDE -> Preprocessor -> Compiler -> Linker -> Loader -> CPU

- volani sluzeb OS zajistuje knihovna RTL
- Z uzivatelskeho pohledu napiseme funkci main a nakonci vratime return 0 (navratovou hodnotu)
    - kdo ale nastavi argc a argv?
    - jak se preda navratova hodnota rodici?

- z pohledu programatora RTL
    1) OS vytvori novy adresni prostor pro proces
    2) OS rozdeli adresni prostor na kod, zasobnik, data, heap, ...
    3) OS nacte executable do adresniho prostoru spolu s potrebnymi knihovnami
    4) OS nastavi CS:IP (EIP) na prvni instrukci nacteneho souboru
        - _start -> crt0

- crt0
    - C znamena "Runtime" a 0 znamena "at the very beginning"
    - zarizuje inicializaci:
        - nastavi argc a argv
        - nastavi stdin/stdout/stderr
        - zavola konstruktory globalnich instanci
        - vycisti bss sekci,etc.
        - pote zavola main (jako jakoukoliv jinou funkci)
        - kdyz main zavola return, crt0 si ulozi navratovou hodnotu a pomoci syscallu pozada OS o ukonceni procesu
            - exit code = return value

- crt0 zasobnik
    - instrukce call a int vyzaduji pouziti zasobniku
        - kdo vytvori zasobnik? OS? crt0?
    - hlavni veci crt0 jsou napsany v assembleru
        - syntaxe dovoluje rezervovat kus pameti pro stack `resb    4096`

- crt0 - debuggovani 
    - pokud je kod prelozen v debug modu, crt0 muze udelat nejake dalsi veci na vic aby se ulehcilo debuggovani
        - muze napr nacist dalsi knihovny
        - precist specificke konfigurace

    - nebezpeci
        - je jednoduche zneuzit program kompilovany v debug-modu -> muze obsahovat zadni vratka
        - napr muzeme nacist fake debug-helper knihovnu, ktera bude delat nekale veci jak chce utocnik

- crt0 - chyby 
    - zadny koned neni bez CHYB

- crt0 - delay
    - call DelayLoop
        - DelayLoop spocita kolikrat musime udelat nic (nop) abychom spotrebovali 55ms (perioda casovaci) -> deleni 55ms nomalizace na 1ms
        - AX, DS citac

- Pametovy prostor procesu
    - System (kernel)
    - zasobnik (env, argc, argv)
    - volny prostor pro zasobnik aby se mohl zvetsovat
    - dynamicke knihovny (malloc.o, printf.o) - lib*, so
    - volny prostor pro heap aby se mohl zvetsovat
    - heap (malloc area)
    - globalni promenne
    - bss sekce (neinicailozaven promenne)- staticky nalinkovane knihovny (+ kod programu - main)
    - startup routine (crt0)

- Spusteni noveho procesu
    - OS prijme pozadavek na zavedeni noveho procesu
        1) UNIX
            - process zavola OS API - fork na vytvoreni noveho procesu
            - tento proces ma ale stejny kod
            - tim padem musime zavolat OS API - exec abychom natahli do adresniho prostoru jiny executable
            - mezi fork a exec muzeme udelat napriklad IO presmerovani
        2) Windows
            - pouziva se pouze jedno OS API - CreateProcess (valny napriklad shellem)
            - CreateProcess delat to co mu je pradany parametrama
        - Vytvori novy PCB a TCB (defaultni vlakno main)

- fork
    ```
    void DoFork() {
        int rc = fork();
        if (rc<0) HandleError();
        else if (rc == 0) ChildExecute();
        else ParentExecute();
    }
    ```

    - OS vytvori totoznou kopii rodicovskeho procesu (ten ktery zavolal fork)
        - virtualni prostor (stack, heap)
        - CPU registry
            - na x86 EAX drzi navratovou hodnotu forku
                - < 0 -> error
                - = 0 -> child
                - = child's PID -> parent
            - file descriptory (soubory jsou sdilene -> provest IO presmerovani je jednoduche)
        - potomek bude mit stejny stav ale kompletne izolovany
        - OS planova je muze naplanovat v libovolnem poradi
            - na Linuxu je child vetsninou prvni

- fork, vfork, clone
    - fork delal deap copy
        - velmi velmi pomale => vfork
        - vfork executoval potomka primo v rodicovskem adresnim prostoru
            - vice nebezpecne, ale rychlejsi nez deep copy
    - clone
        - fork s mnoha parametrama -> vyber toho co chceme kopirovat/delat
            - napriklad jen zasobnik, ne haldu, atd.
            - muze byt pouzit pro fork stejne tak jako pro pthread_create
        - dneska je fork implementovany pomoci clone
    - vylepseni forku - copy on write?

- pthread_atfork
    - programator posila az 3 callbacky
        - prepare - executne se v parentu prod forkem
        - Parent - exekuce parent po forku
        - Child - exekuce childa po forku
     - Proc?
        - Predpokladejme ze existuje child proces ktery prevdal vlastnictvni nejakeho zdroje sveho rodice (locked memory, memory block, ...)
            - Jak korekne uvolnit zdroj a kde?
            - atfork - rodicovsky proces specifikuje kdo a kdy uvolni zdroj
                - implementovani v ramci atfork callback

- PCB
    - datova struktura popisujici proces
    - obsahuje
        - PID
        - PPID
        - state (running, ready, blocked, terminated, ...)
        - registry (kontext procesoru)
        - info o adresnim prosotru (halda, ....)
        - prava se kteryma byl proces spusten (ownership, potom napr ACL - ext4)
        - prioritu
        - tabulka stranek
        - handly otevrenych souboru
            - zavreny pri ukonceni procesu
        - pracovni adresar
        - alokovane zdroje
            - uvolneny pri ukonceni procesu (pokud je neuvolnil sam - bad practice)

 - State Queue
    - kernel si drzi datovou strukturu (linked list) pro stavy procesu, vlaken a dalsich zdroju
    - kazdy stave (blocked, ready, ...) ma svouj linked list
    - v pripade blocked mame jeste fronty ktere specifikuji nad cim je proces blokovanej (klavesnice, sitovka, navratovy kod potomka, ...)
    - kernel presova entity z fronty do fronty 
        - pri spatne implementaci muze byt pomale

- PCB po volani fork
    - potomek ma novy PID
    - potomek muze skoncit s jinym navratovym kodem nez rodic
    - PCB drzi navratovou hodnotu
        - v DELOS je ulozena v EAXu

- Zombie/Defunct Process
    - ukonceny proces ktery furt ma svoji PCB
    - PCB drzi navreatovou hodotu dokud neni prectena
        - napri pomoci IS API call - wait
    - Zombie neni sirotek!!
        - sirotek: rodicovsky proces skoncil driv nez potomek -> adoptace initem
    - Zombie (prirovnani):
        - existujici PCB presto ze proces je davno mrtvej

- jak se zbavit Zombie?
    - kdyz nikdo neprecte exit code, PCB neni uvolene -> memory leak?!
        - out of memory denaial of service?
    - dojdou nam volne PIDy?
    - rodic
        - ifgnoruje SIGCHLD pomoci nastaveni handleru na SIG_IGN
        - jeho potomek se stane zombie
    - alternativne muzeme rodici kill signal
        - pokud rodic odmita precist navratovy kod sveho potomka (to reap the zombie), OS killne rodice a init proces adoptuje nyni uz sirotka - rodicovsky proces uz skoncil
    - init proces periodicky cte navratove hodnoty zombie procesu

- exec
    - nahradi kod aktulne provadeneho procesu jinym kodem
    - nacte novy program
    - nastavy novy zasobnik
        - zmensi pocet vlaken na 1
    - preda kontrolu cr0 nove nacteneho programu
    - uspesny exec nevraci navratovou hodnotu
        - proc? protoze ten kod ktery by ji mel vratit uz neexistuje - prepsali jsme ho
    - PID se nemeni protoze PCB je zachovano

- Kernelovska vlakna
    - spravovany kernelem
        - prepinani, planovani, synchronizace
    - process se muze vykonat nad kernelovskym vlaknem
        - bud 1:1 (heavy) ale jednodue - jeden proces = jedno kernelovkse vlakno
        - nebo 1:N (lightweight)
            - vetsina funkcionality je implementovana v userspacu -> abychom zredukovali pocet zamen CPL

- User-space vlakna
    - RTL knihovna muze pozastavit vlakno, zmenit jeho registry a znova ho spustit
        - tim padem muze prepnout na jine lightweight vlakno bez toho aby se menilo CPL
    - nicmene kernel planovac nevi nic o RTL planovaci
        - muze zpusobit maly vykonon nebo i deadlock (kdyz ty dva planvoace treba nejsou synchronizovane?)
    - reseni - kernel informuje RTL o akcich kere planuje udelat (RTL planovac se podle toho prispusobi)
        - napriklad to ze se kenrel-backendove vlakno zablokuje

- vlakna v Linuxu
    - narozdil od Windows, POSIX a UNIX, Linux nenizna pojem proces/vlakne -> vsechno je task
        - na ne-Linuxovych systemech
            - PCB popisuje co je relevantni pro proces
            - TCB popisuje co je relevantni pro vlakno
    - fork: clone (SIGCHLD, 0)
    - pthread_create: clone (CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND, 0)
        - rodic a potom sdileji stejny adresni prostor, file system, file descriptory a signal handlery

- OOP vlakna
    - OS API CreateThread(proc* func, void *data) vytvori nove vlakno
        - z programatorkseho pohledu se executne od prvni instrukce funkce func
            - Vlaknova funkce je napsana programtorem RTL
            - funkce ma prototyp int func(void *data)
            - data jsou pretypovana a predana jako pointer
            - 
            ```
            int func(void *data) {
                auto thread = reinterpret_cast<CThread*>(data);
                thread->Execute();
            }
            ```

- CreateThreads
    - v realite, OS zacne vykonavat vlastni funci
        - po inicializaci, tahle funkce zavola tu funkce kterou tam predal programator jako parametr
    - time padem func nemusi volat ExitThread OS API protoze ukonceni funkce func znamena vraceni do funkce OS
        - obdoba mainu a crt0