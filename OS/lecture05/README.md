- Bootstrap procesory 0x86
    - po zapnuti pocitaci na najeti BIOSu je aktivni pouze jedno jadro CPU = bootsrap processor BSP
    - OS loader musi zkontrolovat vice jader procesory a aktivovat je
    - Jinak nacte uniprocesorovy kernel (pokud je dostupne treba jen jedno jadro)

- MP Floating Pointer 
Structure
    - zacina s _MP_signature
    - popisuje dostupne jadra procesoru
    - je ulozena but v:
        - prvni kB Extended BIOS Data Area
        - posledni kB (640kB) v conventional memory
        - ROM-BIOS

- MP Config
    - MP Floating Pointer Structure obsahuje MP Config Pointer
    - obsahuje seznam dostupnuch jader procesoru
        - BSP
        - AP (application processor)
    - historicky existovaly single-core SMP systemy

- SMP = symetric multiprocessor

- SMP Bootstrap x86
    1) po zapnutni jsou vsechny jadra v real-modu
    2) BIOS vybere BSP a haltne vsechny ostatni - AP
    3) BSP vykonava kod a hleda \_MP\_
    4) Pokud \_MP\_ strukturu nenajde, nacte uniprocessor kernel
    5) pokud BSP nasel hledanou datovou strukturu, inicializuje APIC daneho BSP
    6) BSP vykonavajici kod "vzbudi" ostatni AP jadra pomoci Ini-IPI (Inter-Processor Interrupt)
    7) AP vykonavajici kod se synchronizuje s BSP
    8) Az se vsechny AP synchronizuji, BSP prepne I/O APIC do symetrickeho I/O modu
        - routovaci tabulka presmerovava IRQ to local APIC
    9) Pokracuj dale ve vlastni inicializaci SMP kernelu

- SMP uskali planovani
    - prestoze SMP rika ze jsi jsou vsechna jadra rovna a vsechna mohou executovat jakekoliv vlakna, neni to dobre tak dedelat protoze:
        - kazde jadro ma vlastni cache - obsahuje data aktualne provadeneho vlakna
        - ucel cache je urcyhleni celeho systemu
        - migrace vlakana z jadra na jadro celkove zpomaluje system

- SMP synchronizace
    - kazde jadro ma pouze jeden zpusob synchronizace -> pouzit atomicke operace

- SpinLock
    - kdyz jadra soutezi o kritickou sekti - potrebujeme pametove misto abychom mohli urcit jestli je prave nekdo v kriticke sekci nebo ne (hodnota ulozena na danem pametovem miste)
    - kdyz je spinlock uzamceny, ostatni jadra cekaji dokud se neuvolni
    - cyklicke cekani
    - nema zadny vyznam kdyz mame jedno jadro (uniprocessor)
    - musime pouzit atomicke instrukce k zamceni a odemceni spinlocku

- inicializace spinlocku

```
spinlock_init:
    jmp spinlock_release
```

- uvolneni spinlocku

```
spinlock_release:
    mov eax, [esp + 4]
    mov dword [eax], 0
```

- zamceni spinlocku

```
spinlock_asquire:
    mov eax, [esp + 4]

spin_wait:
    pause
    test dword [eax], 1
    jnz spin_wait

    lock bts dword [eax], 0
    jc spin_wait
    ret
```

aktivni cekani (zkousime porad dokola) -> nektere zarizeni na to mohou byt nachylne z hlediska energie