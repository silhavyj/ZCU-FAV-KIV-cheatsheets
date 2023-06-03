- Vlakna
    - abychom mohli spustit vice procesu soucasne musime implementovat multithreading
    - pouze se tvari ze procesy bezi paralelne
    - nektere vlakna mohou bezet na jinych jadrech CPU
    - vlakno je jednotka planovani (sekvence instrukci)
    - pro uzivatele to je funkce ktera se tvari ze bezi paralelne k mainu
    - main je defaulti vlakno vytvorene po bootrsapu procesu

- exekuce procesuje je dynamicka neprazdna mnozina vlaken (proces je jen kontejner)

- Time slicing - uniprocessor
    - jedno vlakno bezi na jednom jadre CPU
    - time slicing = rozdelime CPU time na jednotliva kvanta ktera priradime jednotlivym vlaknum
    - prepinani kontextu - kdyz vlaknu vyprsi casove kvantum

- Cooperative Multitasking (non-preemptive)
    - jak velke ma byt casove kvantum?
    - jedeno z moznych reseni je nechat vlakno bezet tak dlouho dokud samo nezavola yield (vzda se CPU)
        - vyhody
            - prepinani kontexu neni casove narocne protoze se to nedeje s kazdym interruptem
            (timer)
        - nevyhoda
            - vlakno muze zabrat procesor na dlouhou dobu -> vypada to jako by cely system zamrzl

- Preemptive Multitasking
    - casovac gneruje preruseni (IRQ0)
    - pri obsluze kernel ulozi kontext daneho vlakna (registry) a nacte kontext jineho vlakna
    - kontext switch musi byt rychly

- Kontext vlakna
    - nektere hodnoty jsou na zasobniku jine v regitrech
    - kontext ulozime
        - napr v DELOS -> do PCB

- TCB (thread control block)
    - registry CPU (kontext)
    - priority
    - stav (runnable, ...)
    - casova razitka (mozno pouzit pri planovani)
    - ukazatel na PCB
    - handlery vyjimek, atd.

- Windows Vlakno uzivatelskeho procesu
    - sklada se z:
        - objekt kernelu (kdy se vytvorilo, stav, prioritu, citac poctu kontext swithu, afinitu, ...)
        - zasobnik
        - TCB, aka TEB aka Thread Environment Block
            - handlery vyjime, error code, local storage (heap), ...

- Stav vlakna
    - zjednoduseni:
        - Created
        - Runnable, Ready (muze se planovat)
        - Running (aktualne je na CPU)
        - Blocked (caka napr na potomka, klavesnici, packet, ...)
        - Terminated
    - realita:
        - Created (nove vytvorene)
        - Odlozene ready (nevim na jake jadro ho dame?)
        - Ready (ve fronte ready daneho jadra CPU)
        - Standby (vybran planovacem; muze se zmenil z hlediska priorit -> pujde zpet do odlozeny-ready)
        - Running (aktualne bezici)
        - Terminated (konec)
        - Waiting (blokovane)
    - prechod z 

- Preemtivni planovani
    - existuje prechod z Runnable do Ready (krome yield)

- Planovani vlaken
    - kernel neplanuje procesy ale vlakna
    - process je jenom kontejner (neprazdna mnozina vlaken)
    - Linux nerozlisuje mezi vlakenm a processem (oboje je to task)
    - vlakna s vyssi prioritou se planuji nejcasteji ale kernel musi zajistit ze nedojde k vyhladoveni
    - vlkane ktere pracuje s aktualnim oknem napr ma vyssi priority -> oblbovani uzivatele ze OS je well-responsive

- RR (round robin)
    - nema priority
    - pouze list (fronta) vlaken
    - time slice ~ 20-50 ms
    - mensi casove kvantum = vetri rezije na kontext switch
    - vetsi casove kvantum = less responsive

- RR s prioritama
    - vice front (= prioritny fronty)
    - jedna fronta = jedna priority
    - vlakna se planuje od prvni neprazdne fronty s nejvyssi prioritou
    - moznost vyhladoveni vlaken s nizkou prioritou
        - porad vybirame vlakna s vysokou prioritou (napr neustaly tisk)

- Planovac v Linuxu
    - vlakna s vyssi prioritou dostanou vice casoveho kvanta
    - casve kvantum vlakna se zmeni podle toho jak moc vyuzivalo CPU
    - snaha nechat bezet stejne vlakna na stejnem jadre CPU
    - kazde jadro CPU ma vlastni task-queue
        - pokud je tato fronta moc velka, kernel ji rozdeli mezi vice jader
    - kazde vlakno ma afinitu (na kterych jadrech muze bezer) - bitmask
    - Completely Fair Scheduler

- Zaklady planovani
    1) vezmi frontu vlaken s nejvyssi prioritou (neprazdnou)
        - existuji 2 typy front (active & expired) - kazda znich ma 140 statickych prooorit?
            - 0 - 99 real-time tasks
            - 100 - 139 - normal tasks
    2) vezmi prvni vlakne z te fronty a spocitej casove kvantum ktere mu bude prideleno 
    3) naplanuj ho (dej ho na CPU) a presun ho do expired fronty
    4) goto 1 

- Vypocet casoveho kvanta
    - SP = staticka priorita
        - SP < 120 -> kvantum = (140 - SP) * 20
        - SP >= 120 -> kvantum = (140 - SP) * 5
    - mensi SP = vyssi priorita
    - DP = dynamicka priorita
        - odpovida chovani procesu v posledni dobe
        - cas ktery vlaskno nevyuzilo v ramci prideleneho casoveho kvanta
            - vlakno dostane bonus <0, 10>
                - 0: sniz prioritu o 5
                - 5: nic
                - 10: zvys prioritu o 5
    - DP = max(100, min(SP - bonus + 5, 139))