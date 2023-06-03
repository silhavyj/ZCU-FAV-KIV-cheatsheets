- Interni synchronizace procesu/vlaken
    - interrni synchronizace vlaken?
    - v Linuxu, PCB reprezentuje vlakno, v jinych os TCB reprezentuje vlakno
    - jednota planovani
        - kernel-sheduled entities

- Semafor
    - abstraktni datovy type ktery ridi pristup ke sdilenemu zdroji
    - sdilen mezi vice vlakny
    - nema zadne info o tom co ten sdileny zdroj je (tiskarna?, pole?)
    - obahuje:
        - limit - kolik vlaken muze pristoupit k danemu zdroji bez toho aby se zablokovaly
            - binarni semafor ma limit 1
        - citac kolik vlaken ma aktualne pristup ke sdilenemu zdroji
        - frontu blokovanych vlaken

- Acquire
    - aka wait, down, or P
    - vlakno se zepta jestli muze "vsoupit do semagoru" a citac semaforu se snizi o n (n je typicky 1)
    - OS musi zajistit ze Acquire je atomicka operace
        - test jestli po dekrementace scitace je hodnota stale kladna
        - pokud ano tak se hodnota citae snizi (vlakno se neblokuje)
        - pokud ne, hodnota citace zustane stejna a proces se zablokuje

- Acquire – Uniprocesor
    - Acquire musi byt atomicke -> OS musi implementovat ochranu kriticke sekce (zmena hodnoty citace)
    - na uniprocesoru se to muze zmenit naprimo
        - ALE musime zamezit kontext switch (napr docasne ignorovat tiky casovaci IRQ0)

- Acquire – Multiprocesor
    - na multiprocesoru nestaci pouze zamaskovat interrupty pro jedno jadro -> museli bychom to udelat pro vsechny jadra daneho CPU
        - potrebovali bychom dodatecnou synchronizaci
    - priznak by musel byt globalni pro vsechna jadra
    - zmena hodnoty citace je SPINLOCK-like code

    ```
    void sem_acquire() {
        while (counter == 0) {
            block_thread();
        }
        spinlock_lock(&lock);
        counter--;
        spinlock_lock(&unlock);
    }

    void sem_release() {
        spinlock_lock(&lock);
        counter++;
        spinlock_lock(&unlock);
        nofify_threads();
    }
    ```

- Asquire - blokovani vlakna
    - pokud by citac nebyl kladne cislo -> OS musi vlakno zanlokovat
        - stav vlakna se zmeni na blokovany
        - kernel prida referenci na PCB do fronty vlaken cekajici na konkretni semafor
        - kdyby bylo n > 1 -> bylo by to prilis komplexni (museli bychom pridal vyzadane n)

    
- TryAcquire
    - misto blokovani - budto se nam podari uspet nebo vratime error code

    ```
    int sem_try_acquire() {
        if (counter == 0) {
            return 1;
        }
        spinlock_lock(&lock);
        counter--;
        spinlock_lock(&unlock);
        return 0;
    }
    ```

- TryAcquire - SpinCount
    - diky atomickym instrukcim muze byt spinlock implementovan v userspacu
    - vlakno muze soutezit o semafor s prededfinovanym poctem pokusu nebo timeoutem (nemusime menit CPL)

- Release
    - aka signal, up, or V
    - vlakno opusti kritickou sekci v semaforu - citace se inkrementuje o n (n = 1)
    - po inkrementaci citace - kernel musi vzbudit jedno ze spicich vlaken blokovanych nad danym semaforem

- Release - vzbuzeni procesu
    - naivni pristup
        - z neprazdne mnoziny procesu blokovanych nad danym semaforem vzbudime prvni flakno
        - presuneme TCB do runnable, a zmenime jeho state
    - Ale?
        - co kdyz vlakno zavolalo Asquire s n > 1?
        - Co kdyz je vlakno blokovano jeste na necem jinem (e.g. klavesnice viz DELOS)

    - co kdyz vlakno zavolalo Asquire s n > 1?
        - musime najit vlakno ktere je blokovano s n <= citac semaforu
        - pokud ale existuje vlakno s jeste mensim n -> kere se spusti prvni?
            - museli bychom iterovat pres danou frontu a hladat vhodne n -> pomale
            - proto pouzivame n = 1
    - Co kdyz je vlakno blokovano jeste na necem jinem
        - napriklad klavesnice, debugger, sitovka
        - vlakno nemuze jit hned do stavu runnable
        - kernel musi vybrat jine vlakno, ktere muze jit primo do runnable (z fronty blokovanych vlaken)
        - naivni pristup
            - spust vsechny runnable-ready vlakna
            - ty krese neuspeji se hned zase zablokuji
            - musime pouzit WHILE a ne IF v Acquire!!!
            - moc komplexni
            - probudime jen prvni runnable-ready

- Producent - Konzument:
    - buffer omezene velikosti buff[n];
    - 1 semafor inicializovan na 0 (cteni) - R
    - 1 semafor inicializovan na n (zapis - n  - polozek) - W
    - 1 semafor nastaven na 1 (vzajemne vylouceni) - S

    - producent:
        ```
        P(W)
        P(S)
        push(x)
        V(S)
        V(R)
        ```

    - konzument:
        ```
        P(R)
        P(S)
        pop(&y);
        V(S)
        V(W)
        zpracuj(y)
        ```

- Mutex
    - vypada "skoro" jako binarni semafor ale neni stejny
    - ma koncept vlastnictvi
    - muze byt odemknut pouze tim vlaknem ktere ho zamklo
    - muze zabranit nasilnemu ukonceni vlakna (v pripade ze vlakno drzi zamek - mutex)

- Pipe
    - buffer se dvema konci (in, out)
    - file descriptor pro kazdy konec
        - WinAPI podporuje duplex pipy
    - muze byt pojmenovana -> pristupne z file systemu
        - napri zapis do terminalu viz DELOS
    - pouzita pro presmerovani stdin, stdout, stderr procesu
        - poslat vystup na vstup jineho procesu
        - poslat error do souboru atd

- Pipe - cteni & zapis
    - buffer pipe ma fixni velikost
        - muze byt specifikovano pro inicializaci
    - read & write vlakna musi byt limitovany na velikost bufferu
    - aplikace pipe na problem producent-konzument
        - kruhovy buffer
        - konzment cte (popuje) n bytu
        - producent zapisuje (pushuje) n bytu
        - uz zname reseseni
            -  musime je vyresit situace kdy se zapisuje/cte vic bytu nez je dostupno
            - vice producentu/konzumentu
            - Duplex roura
            - je vhodne mit vice implementaci specialne upravene pro konkretni pripady
                - optimalizovana simplex pipe pro 2 vlakna
                - duplex pipa

- stdin, stdout, stderr – Linux
    - s POSIX, zavri pozadovany handle 
    - OS potom pouzije stejne cislo pri vytvareni noveho resp duplikovani existujicho (interni create) file descriptoru
    ```
    dupl2(fileno(new_stdin_opened_file), STDIN_FILENO);
    dupl2(fileno(new_stdout_opened_file), STDOUT_FILENO);
    dupl2(fileno(new_stderr_opened_file), STDERR_FILENO);

    flose(new_stdin_opened_file);
    flose(new_stdout_opened_file);
    flose(new_stderr_opened_file);
    ```

- stdin, stdout, stderr – WinAPI
    - CreateProcess prebira jako parametr strukture ktea definuje parametry daneho procesu -> flagy, stdin, stdout, stderr.

- Zpravy (messages)
    - MS Windows princip synchronizace
    - pro UI jsou povinne
        - kazdy vizualni prvek je okno kere prijima zpravy
        - GUI ma hlavni smycku kde prijima zpravy (WindowProcedure)
            - konzolova aplikace to muze ale nemusi implementovat
    - jedno vlakno posila zpravy jinemu vlaknu
        - muze ale nemusi cekat nez si prijemce zpravu precte
    - OS spravuje frontu zprav pro kazdy proces
    
    ```
    int wmain() {
        CreateWindow(….)
        while(GetMessage( &msg, NULL, 0, 0 )) {
            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }
        return msg.wparam;
    }
    ```

- Posilani zprav
    - PostMessage neni blokujici -> nevraci se navratova hodnota
    - SendMessage JE blokujici -> blakno se zablokuje dokud si prijemce zpravu neprecte
        - odesilatel si pote precte navratovou hodnotu
    - WM_COPYDATA - specialni kod zpravy
        - IParam je ukazatel na blok pameti ktery je read-only
        - pokud prijemce chce tyto data -> musi si udelat deepcopy
        - sdileni pameti
        - muze mit lifetime

- PostMessage - implementace
    - OS zavola WindowProcedure, ketra serializuje prijate zpravy
        - OS smaze zpravu z frontu zprav
    - OS prida zpravu do prijemcovi fronty zprav
    - odesilatel se neblokuje (neni navratova hodnota)
    - OS smaze zpravu z fronty zprav daneho vlakne tesne pred tim nez zavola WindowProcedure (ketere zpravu zpravuje)

- SendMessage - implementace
    - musime pouzit synchronizacni primitivum spojene s danou zpravu pro blokovani odesilatele dokud si prijemce zpravu neprecte
    - odesilatel si cte navratovou hodotu
        - binarni semafor staci (na implementaci)
    - kdyz prijemce opusti WindowProcedure, CS:EIP se vrati do kodu OS (jadra) ktere prekopiruje EAX prijemce do EAXu odesilatele

- Signaly
    - dalsi zpusob POSIX synchronizace
    - handler signalu je definovan pomoci id (index signalu)
        - OBDOBA IDT :)
    - napriklad ctr+C v konzoli
        - OS preformuluje ctr+c na SIGINT (signal na preruseni procesu) a naplanuje vykonani prislusneho handlu
        - co pak nasleduje zalezi na konkretnim handleru daneho signalu

- Signaly - defaultni handlery
    - co kdyz se OS pokusi vykonat dany handler a on tam neni?
        - bud segmentation fault
        - nebo exekuce random kodu -> povede k vyhozeni vyjimky (memory corruption, ...)
    - OS vzdycky nastavy defaultni handlery
        - stejne jako napr BIOS poskytne IDT tabulku (0x00 - real-mode)
    - pokud process handler neprepise vlastnim, OS pouziva defaultni

- Signaly - user handlery
    - napriklad defaultni SIGINT handler ukoncuje procs
    - Co kdyz napriklad proces kopiruje soubor?
        - proces by mel presut kopirovani a soubor zavrit
    - proces si nainstaluje vlastni SIGINT handler ktery pak dela ocekavane preruseni kopirovani a uzavreni souboru

- Ignorovani signalu
    - proces muze ignorovat vetsinu signalu
    - nikdo je pak nebude vykonavat (ani proces ani kernel)
    - dva signaly NEJDOU ignorovat
        - SIGKILL - ukonceni procesu (termination)
        - SIGSTOP - zastaveni procesu

- Procesne rizene signaly (Process-Directed Signals)
    - POSIX rika ze vsechny vlakna na strane jadra musi mit stejne PID
    - aby se vykonal signal handler, OS vybere jedeno vlakno ktereho vykona - Process-Directed signal
    - pri pouziti pthread_sigmask thread (Linux task) muze ignorovat signaly pro sebe a vsechny sve potomky
        - tim padem je mozne handlovat vsechny signaly ve specifickem vlakne
        - mnozina vlaken mapovane na vlakno jadra (stejne PID) se redukuje pouze na jedno vlakno
        - pokud je tato mnozine prazdna, OS ukonci proces

- Vlaknove rizene signaly (Thread-Directed Signal)
    - funkce kill prebira cislo signalu jako jeden z parametru (napr SIGKILL)
    - funkce pthread_kill posle signal konkretnimu vlaknu aby se ukoncilo
        - napr SIGSEGV is poslani vlaknu ktere zpusobilo invalid memory access
        - pokud handler toho signalu neexistuje, OS ukonci cely procel
        - => ukonceni celeho procesu muze byt zpusobene chybejicim handlerem jednoho vlakna

- Signaly - fork, exec
    - fork: potomek dedi handlery signalu sveho rodice vcetne tech ktere jsou ignorovany
    - exec: prepise existujici handlery -> OS nastavi defaultni handlery

- Implementace signalu
    - signaly per process/thread
        - uzivatelske handlery (prpsani defaultnich)
        - spinlock osetruje pristup k handlerum
        - 64 bit maska definuje ignorovane signaly
        - fronta signalu ktere se maji zpracovat

- Real-Time signaly
    - 32 (0-31) standardnich signalu (preddefinovany ucel)
    - 32 (32-63) real-time signaly (nemaji preddefinovany ucel)
    - "real-time" - obslouzeny ASAP
    - rozdil "real-time" oproti normalnim signalum
        - vice instanci stejneho signalu muze byt cekajici na oblouzeni
        - jejich handlery jsou aktivovane v zarucenem poradi
        - obsahuju pridane systemove informace

- Signaly vs Zpravy
    - Linux ma 64 signalu navrzene pro konzolove programy
    - Windows GUI cte zpravy
        - WM_QUIT slouzi (ma stejny ucel) jako SIGKILL
    - Windows ma 2^sizeof(int) zprav navrzeni pro GUI aplikace
    - Linux GUI pouziva jiny mechanismus - signals & slots (QT)