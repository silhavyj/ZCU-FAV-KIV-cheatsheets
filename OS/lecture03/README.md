- Monoliticky kernel
    - CPL0
    - Drivery mohou bezet v CPL0
    - typickou soucasti je FS
    - drivery a aplikace tretich stran v kernelu prestavuji riziko
    - "jeden velky kus programu"
    - napr Linux

- panic
    - kernel selze -> nevi se co dal
    - kernel je typicky stabilni
        - drivery v CPL0 seboou muzou sundat cely system
        - nedejboze chyba HW

- Mikrokernel
    - Chceme co nejvic veci odebrat z CPL0
    - Tim padem zlepsime stabilu systemu
        - restart kodu co selhal mimo kernel
    - v jadre akorat:
        - sheduler
        - alokator pameti
        - IPC
    - v pripade naivniho pristupu pomaly - moc contex switchu
    - kernel prijima syscally a proposila je danym serverum napr FS, atd

- Hybrid kernel
    - pul na pul
    - napr Windows
    - osuneme potencionalne nestabilni veci do CPL3
        - drivery tretich stran

- CPL0 - jadro
- CPL1 - hypervizor
- CPL2 - drivery
- CPL3 - uszivatelske programy

- RTL
    - runtime library
    - getTickCount, pocet tiky procesoru of resetu CPU
    - Neni tak primocare u vice jadroveho CPU
    - instrukce RTDTR dovoli ziskat hodnotu s CPL3 -> netreba resit pres syscall

- Dynamic Libraries
    - skoro kazdy program pouziva knihovny
    - staticky linkovana
        - vezmu knihovnu a pribalim ji do binarky
        - bez externich zavislosti
        - horsi aktualizace pokud vyjde update (musim prekompilovat)
        - velikost
    - dynamicky linkovana
        - knihovna je mimo binarku
        - ELF soubor definuje jake funkce potrebuje pro svuj beh
        - OS namapuje a resolvuje adresy dle pozadavku v ELF souboru

- Dynamicke nacitani
    - proces rekne OS kdy chce nacist dll (so)
    - Dlopen Linux, LoadLibrary WinAPI
    - OS pri prvni pozadavku zavede knihovnu do pameti a drzi si counter, pri kazdem odmapovani counter--, pokud counter = 0, odmapuju knihovnu

- Dynamic GetAddress
    - ziskani adresy funkce podle jejiho nazvu
    - programator musi znat signaturu
    - GetProcAddress - Windows
    - Dlsym - Linux

- DLL init
    - po zaveni knihovny OS zavola jeji funkci main
    - data jim tim moznost se nainicalizovat
    - knihovna je jen dalsi ELF soubor

```
HMODULE lib LoadLibrary("lib.dll");
Tfunkc *func = GetProcAddress(function");
Func();
FreeLibrary(lib);
```

- Parametry predane kernelu
    - pres registry (omezene mnostvi)
    - pres zasobnik
        - v DELOS napr _switch_context(regs)

- kernel entr
    - IDT index
        - nutne nelezi na 0x0 ale tam kam ukazuje IDTR
    - nastavi flagy
    - kernel by nemel verit stacku processu
        - muze byt pouzito pro utok

- Interrupt
    - minimalni prace co je treba udelat (viz top-half)
    - musi byt rychly
    - pokud nestihnu rychle, klidne preplanuji a proces bude cekat co pracuje jiny

- pomoci segmentovych registru menim CPL?

- TSS
    - z user space do kernelu - drzi adresu kernolovskyho stacku
    - muze byt pouzit pro odkladani kontextu
    - muze byt vytvoren pro kazdej proces
    - depricated x86-64

- syscall a sysret nahrazuji int a ret
    - modernejsi a rychlejsi
    - 3x rychlejsi

- Vyjimky
    - Trap (brakpoint)
    - Fault (zotavitalna chyba - napr page fault)
    - Abort (nezotavitelna chyba napr double falt)

- double fault
    - pokud napr nastane page fault a neexistuje handler -> vznikne double fault

- tripple fault je pokud neexistuje hendler double faultu

- deleni 0
    - bez interruptu (vyjimek) by se PC restrartoval
    - pokud thread nema handler, proces bude killed
    - pokud ma handler, kernel ho executne
        - to jestli ma nebo nema handler se zjisti z TCB
    - try/catch

- VESA
    - video electronics standard association
    - kdyz nestaci 64kB na video, rozsirime vide podle potreby
    - VESA standard zjisti od monitoru jaky vyzaduje rozliseni a kolik stranek bude na to potreba -> reseni pres strankovani aby dokazal zobrazit vsechny pixely
    - VESA-linar buffer
        - primy pristup do pameti - rychly
        - pres VESA asi ziskame addresu bufferu

- chained ISR
    - vymenit IDT adresu za svoji
    - musime znovy pusnout flags
    - zavolame puvodni adresu ISR
    - IRQ (drat) do PIC
        - asynchronni preruseni
        - IRQL - priorita
        - PIC prevadi cisla IRQ na indexy do IDT
        - IRQ0 - timer
            - tika kazdych 55ms (pokud neni nastaveno jinak)

- IO
    - prevna predem znama adresa na sbernici
    - komunikujeme s radicem daneho zarizeni
    - komunickace pres jeho registry (ridici, datovy -> zname adresy)
    - nahrazeno MMIO

Virus
    - DOS dovoli uzivateli delat co chce - treba prepsat MBR

- Vypocet (resolvovani) address u dynamicky nalinkovani knihovny
    - Chci zavolat puts - nevim kde je (je soucasti libc)
    - Zavolam PLT[n] - Procedure Linkage Table
        - n prestavuje index ktery odpovida funkci puts
        - PLT[n] obsahuje pouze 3 instrukce
            - jump to GOT[n]
            - prepare resolver (napr push 0x00 - index funkce puts napr)
            - jmp to PLT[0] - resolver puts
    - Pote co se resolvuje adresa puts ulozi se do GOT[n] (Global Offset Table) -> drzi nazvy funkci, promenny atd, a adresy kde jsou namapovany
    - pri dalsim volani PLT[n] -> GOT[n] uz se navratime zpatny do PLT[n] kvuli resolvovani ale uz rovnou skocime na prislusnou adresu funkce puts 
    - lazy resolving!