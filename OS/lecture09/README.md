- Motivace
    - Predpokladejme ze mame spcificky OS a CPU
        - program kompilovany pro tento CPU a jeho efektivni operacni mod funguje nativne
        - co kdyz ale potrebujeme spustit program, ktery je zkompilovany pro jiny mod CPU (real/protected)? nebo pro jiny CPU (arm/intel) nebo jinou architekturu?
        - potrebujeme but emulaci nebo virtualizaci

- Emulace
    - SW prostredky vytvorime iluzi realneho HW
    - napr muzeme spustit DOSBox na ARMu
        - -> spustit legaci x86 programy na ARMu
        - muzeme spustit neco co potrebuje typ HW ktery ale nemame
        - univerzalni ale vypocetne narozne reseni
            - emulace muze podporovat virtualizaci

- Virtualizace
    - virtualizaci nedela emulaci HW ale pouziva realny HW
        - vetsi vykon nezemulace
        - limitovane mnozstvi kompilovanych programu pro dany HW

- Hypervisor
    - aka Virtual Machine Monitor (VMM)
    - (Host) vytvori a executne virtualni stroje (guests)
    - Typ 1 - executne se primo na HW
        - Xen, VMWare EXS, Hyper-V
    - Typ 2 - hostovany operacnim systemem
        - VirtualBox, WMVare Player

- Mainframe
    - mainframe dominoval pred erou pocitacu (50-70 leta)
        - vyznacovaly se velkym vykonem (tehda)
    - nebyly mezi sebou kompatibilni
    - jak spustit SW pro jeden mainfram na jinem?
        - IBM oddelila (virtualizovala) architekturu a implementaci
        - tim padem by mozne spustit vice OS na jednom mainframu

- PC
    - mnohem mene vykone a spolehlive nez mainframe
    - ale zase je levnejsi nez mainframe
        - pomer cena vykon zpusobil ze PC byly vsudypritomny
    - pouziti vice PC umoznilo vytvorit distribuovany system aka cluster
        - stale levne v porovnani s mainframem
        - softwarove implementova spolohlivost
        - s pouzitim virtualizace muzeme v takovem klastru efektivne vytvori a spustit virtualni stroje 
            - aka cloud :)

- V86 - Motivace (1)
    - bylo nebylo v kralovstvi x86...
        - zpomeli si na prvni prednasku o MS-DOSu
    - programatori prevzdali protected-mode kluvi velkemu adresnimu prostoru a izolaci procesu
    - Nicmene...
        - nektere programy nebyly zkompilovane pro protected-mode, ale potrebovali jsme aby je bylo mozne spustit
    - bylo zapotrebi ridit zarizeni
    - Motherboard, graficka karta, sitovka - vsechny maji svuj vlastni BIOS
    - BIOS motherboardu inicializoval zarizeni a zpnul x86 v realnem rezimu -> BIOS obsahoval programy kompilovane pro realny mod

- V86 - Motivace (2)
    - Spolu s inicialize, BIOS poskytuje rutiny pro rizeni HW
        - prepinani VESA stranky na SVGA 
        - casto vyuzivane ovladacem zarizeni (je jednodusi pouzit neco co uz je napsane v real-modu nez si psat vlasni v protected-modu)
    - Kvuli velkym rozdilum mezi real modem a protected modem nemuzeme spustit real-mode program primo v protected-modu
        -> x86 se musi prepnout do virtualniho modu V86

- V86 - popis
    - dostupne od 80386
    - HW virtualizace 8086
        - pouziva segmentaci jako realny rezim -> 20 bitova adresace ale podlehaji strankovani v protected-modu
    - long-mode spusti 8086 program s VT-x

- V86 - princip
    - kdybychom prepnuli procesor z protected modu do realneho modu, ztratili bychom vsechny pamet a OS by selhat
    - Tim padem
        - vytvorime virtualni adresni prostor o velikosti 1MB pro real-mode programy s CPL3
        - pro nej vytvorime monitor - protected-mode task s CPL0
        - pouziti privilegovane instrukce real-mode programu vyhodi vyjimku
            - pote jadro vykona ISR poskytnute monitorem
        - kdyz real-mode program zavola legacy OS API, pouziva int
        - monitor volani odchyti zkrze ISR a pretransformuje ho na OS API call hostu
        - vysledkem je ze kernel muze executnout real-mode privilegovane instrukce tak jak si preje bez toho aby real-mod program o tom neco vedel

- V86 - Aftermath (nasledky)
    - program se muze vykonavat rychle protoze CPU vykonava instrukce naprimo
        - krome privilegovanych instrukci neexistuje zadne zpomaleni
    - Neni nutne pravda ze real-mode program nema zadne prostredky na to zjistit ze je virtualizovan
        - nejake instrukce mohou proflaknout nejake info, pokud tedy neni pouzit VT-x nebo jsou instrukce vymazany z kodu pred jeho exekuci
        - starsi programy by nic nezjistili protoze takove instrukce jeste ani neexistovaly

- Two-OS Problem
    - Jeden hostovany OS nemuze ublizit jinemu hostovanemu OS
    - privilegovana instrukce - CPU vyhodi vyjimku pokud instrukci vyzadovne CPL NEodpovida guest's CPL
    - citlive instrukce meni HW konfiguraci a jeji vysledek zalezi na aktualni HW konfiguraci

- Privilegovane vs citlive
    - efektivni a bezpecna virtualizace vyzaduje ze kazda citliva instrukce je zaroven privilegovana
        - napr mnozina cislivych instrukci je podmnozina privilegovanych
        - dokud neprisel VT-x a AMD-V, x86 nesplnoval tento pozadavek
    - CPU vykonava neprivilegovane instrukce primo
        - zadna ztrata vykonu
    - privilegovane (a citlive) instrukce vyhodi vyjimku
        - kernel ISR ji obslouzi
        - tyto instrukce musi byt emulovane - emulace uvnitr virtualizace
            - ztrata vykonu, ale nutna

- Paravirtualizace
    - bez HW podpory jako je VT-x musime modifikovat guest's code aby nepouzival citlive ale neprivilegovane instrukce
    - guest OS by mel o hypervisoru
        - hypervisor sedi mezi HW a OS
        - virtualizovany OS pouziva API hypervisoru pro zvyseni vykonu
        - rika hypervisoru co chce delat
        - a hypervisor presmerovava urcite krivicke ukoly hostovanemu OS
    - Blizime se k vykonu plne vyrtualizace ale...
        - co kdyz by OS spustil program ktery obsahuje zakazane, nebezpecne instrukce? -> security hole?

- Binary Translation (binarni preklad?)
    - predtim nez spustime gues code, analyzujeme ho
    - nahradime nebezpecne instrukce bezpecnymy
        - blok o m nebezpecnych instrukci muze byt nahrazen blokem o n bezpecnych instrukci
    - je takve mozne nahradit bloky kodu rychlejsimi
        - napriklad pokud privilegovana instrukce vyhodi vyjimku -> vedeto na emulaci
            - tim padem ji muzeme nahradit primo emulovanym kodem -> vyheneme se preruseni
    - nahrazovani instrukci neni jednoduche!
        - nahrazovane bloky instrukci mohou mit jinou velikost, moho obsahovat JMP instrukce, adresy promennych atd

- Binary Translation - int 10h
    - uvazujme real-mode program ktery se pokusi meznit mod obrazovky na textovy mod CGA 80x25
        ```
        mov ax, 3
        int 10h
        ```
    - takovyto kod vyzaduje emulaci graficke karty
        - tim padem ho muzeme primo nahradit volanim CGA emulace bez toho abychom prepinali kontext CPL0
        ```
        mov ax, 3
        pushf
        call EmulatedISR10h
        ```
- Binary Translation - JMP
    - predpokladejme ze mame smycku s privilegovanou HTL (intrukce) ktere haltene jadro dokud neprijde externi interrupt
    - HLT vyzaduje CPL0, je ji ale problematicke nahradit protoze je pouze 1B dlouha (call instrukce je alespon 2B dlouha - 2B op code a 1B relativni adresa)
        - nahrazovani htl instrukce neni snadne

- VT-x
    - cilem je eliminovat potrebuje paravirtualizace a binary translation
    - CPU vykonava kod v dvou novych modech - VMX root a non-root
        - eliminuje potrebu hyporvisoru executovat v CPL0 a kernel s CPL1
        - pocet prechodu mezi temito mody by mel byt minimalni
            - binary translation jiz neni potreba
        - prepnuti do VMX root skrze VM Entry
        - prepnuti do VMX non-root skrze VM Exist (guest OS)
    - hostovany OS executuje bez zadnych dalsich modifikaci (narozdil od paravirtualizace) v VMX non-root modu
        - hostovany os nema zadny stavovy bit aby zjistil ze je virtualizovan
        - kdyz se pokusi o nebezpecnou operaci, jadro spusti VMX transition
            - hypervisor prevezme kont4rolu a udela co je potreba v VMX-root modu

- TLB
    - cisteni TLB pri kazdem VMX translation by bylo bezpecne ale pomale
        - 1 generace VT-x to tak delala
        - aby se zvysil vykon, VMX non-root guest ma vode VPID (Virtual processor ID)
            - hypervisor priradi VPID, host ma 0
        - jadro nedovoluje guest OS pristoupit TLB zaznamy ktere nespadaji pod jeho VPID

- Paging
    - Host OS pouziva shadow page table
    - Bez VT-x, gues'ts page table by byla read-only
        - s pouzitim COW? zachytime zapisy a synchronizujeme shadow table
        - host oblbne guesta ze si bude myslet ze ma kontrolu nad pozici tabulky
    - Cele je to pomale tim jak COW princip jde zkrz ISR
    - Reseni, VT-x pouziva zanorenou/rozsirenou tabulku stranek - rychlejsi
    - Guest pouzivate tabulku stranek stejne jako predtim - CPU preklada fyzickou adresu prostrednictvim rozsirene tabulky stranek - zanonere do fyzicke adresy hostu

- VT-d/VT pro prime I/O
    - take I/O impikuje snizeni vykonu
        - take diky VMT prekladu do VMX-root
    - VT-d  dovoluje mapovani IRQ a rozrirene DMA specifikace
        - dovoluje priradit konkretni zarizeni prislusnemu hostu (napr USB)
        - vyzauje rozsireny xAPIC

- Virtualization for Aggregation VfA
    - narozdil od spoustenim vice guestu na jednom pocitace, protojime vice pocitace ("superpocitac")
    - vyvorit pocitace s velkou RAM a velkym pocet CPU jader je relativne levne
    - Guest OS bude jen virtualni SMP

- VfA - jak to udelat?
    - na kazdem vSMP uzlu bude VMM ktery pouziva napr VT-x a komunikuje s ostatnima VMM

    - kdyz presusime (zachytime) pristup ke zdroji ktery se nachazi na jinem uzlu, pouzijeme zpravy na presmerovani pozadavku na jinej uzel

- Rootkit
    - jak tezky by bylo vytvorit rootkit kdyz mame virtualizaci?
    1) inicializuj VT-x
    2) vytvor VM a VMM
    3) nakopiruj host OS do VM
    4) predej rizeni VM
    5) ukonci puvodni host OS (ted bezi v 
    VM)
    6) pri prakladu VMX na VMX root, rootkit se aktivuje a vi vsechno co se dele ve VM guest OS

- Detekovani rootkitu
    - ackoliv neexistuje oficialne dokumentovany status bit ktery indikuje to ze je system vyrtualizovan
    - CPUID vzdy dokuni VM se ukoncit
        - tim padem ma mnohem vetsi zpozdeni s aktivnim VT-x