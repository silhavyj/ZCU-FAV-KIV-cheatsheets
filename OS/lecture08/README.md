- Konfigurace routeru
    - pripojeni konzole routeru do serioveho portu pocitace
        - BIOS poskytuje zakladni praci se seriovymi porty
        - tohle je ale real-mode - zadna izolace procesu
        - x86 musime komunikovat po seriove lince - in, out instrukce -> musime pouzit CPL0
        - time padem pouze kernel komunikuje a provadi virtualizaci seriove linky jednotlivym procesum
        
- Tiskarna
    - PC pouzivali LPT (paralelni port) pro pripojeni tiskarny
    - vice procesu pouziva LPT ve stejny cas -> mixovani prikazu -> spatny print
    - OS virtualizuje tiskarnu (LPT) aby zajistil atomicitu jednotlivych tisku

- Disk
    - co kdyz vice vlaken chce cist ze stejneho disku?
    - stejny problem jako s tiskarnou a seriovym portem
    - stare rotacni disky
        - OS muze zmenit poradi pozadavku aby usetril cas (rotace disku)

- VGA
    - rizeni probiha pres interrupty a porty
    - device-controller register je vystaven na predem znamou adresu
    - zmena grafickeho modu
        ```
        mov ax, 13h
        int 10h
        ```
    - zmena jedne barvy v palete barev
        ```
        outb(0x3c8, 1); // color index in the palette
        outb(0x3c9, 255); // red
        outb(0x3c9, 0); // green
        outb(0x3c9, 0); // blue
        ```

- VGA cekani na paprsek
    - CRT techonologie
    - muzeme vykreslit az tehdy kdyz se elektronovy paprsekt vrati zpatky na zacatek monitoru
    - frekvence vraceni paprsku je prilis rychla na to abychom pouzili IRQ -> pouzijeme poll 
        ```
        do {
            while (inportb(0x3DA) & 8); //vertical retrace in progress
            while (!(inportb(0x3DA) & 8)); //refreshing display
            Draw_Next_Animation_Frame(); //draw to VGA buffer
        } while (there_are_frames_to_draw);
        ```

- Blokova a znakova zarizeni
    - blokova zarizeni presouvani data na/z HW po blocich
        - napriklad disk - cteni/zapis celych sektory
        - chceme urychlit IO
        - OS cachuje cele bloky
    - znakove zarizeni presouva data na/z HW po bytech
        - seriova komunikace
        - zacna cache
    - kazde zarizeni ma sve ID
        - ID je strukturovane -> muzem zaradit zarizeni stejneho druhu do jedne skupiny

- I/O API
    - promator nepracuje prime s HW ale vola API OS
        - napriklad read/write (pouziva jenom file descriptor)
        - nezajimaji ho HW specifikace
        - vyuziva virtualni FS
            - UNIX-like: VFS
            - Windows: Installable File System
    - sektor disku muze byt ulozen na lokalnim disku, virtualnim disku, RAM, nebo sdileny po siti

- EXT2
    - pouziva inody
    - inody neobsahuje nazev souboru!! (to je az v adresari)
    - superblock, inode bitmap, clusters bitmap, clusters
    - inode obsahuje metadata (velikost, parent inode, ...)
    - prime odkazy, neprime odkazy, dvojite neprime odkazy
    - slozka: drzi dvojice cislo inodu, nazev
    - nepodporuje ACL (read/write/execute pro daneho uzivatele ne jen owner:group)
    - rychle pro male soubory (vejsou do primych odkazu)
    - jen pro Linux (Milan to nevedel...)

- FAT
    - formatovany disk je rozdelen na clustery
    - boot sector | FAT1 | FAT2 | Root dir | clusters
    - soubor muze byt fragmentovany (nevyhoda - hejbeme hlavickou disku)
    - fat je defakto tabulka indexu odkazujici na nasledujici cluster ktery je soucasti souboru
    - fat[i]
        - obsahuje but index dalsiho clusteru
        - nebo EOF

- NTFS
    - nastupce FAT (neobsahovala zurnal)
    - cely disk -  MBR | Reserved | VBR | $BOOT | $MFT | Files 
    | Reserved 
    - NTFS partition - VBR | $BOOT | $MFT | Files 
    - pouziva zurnalovani
        - zaznam akci co se chystam udelat nez je udelam
    - soubor ulozen po fragmentech
        - file1 | 0 | 4096
        - file2 | 4 | 1024
        - file4 | 5 | 2048
    - VBR = volume boot record
        - primarny boot kod, nacte bootloader $BOOT
    - MTF (master file table - seznam fragmentu - soubory) zaznam je <filename, start_cluster, size, cluster_count>
        - size kvuli poslednimu cluteru
        - kopije MFT je na konci disku
    - podporuje ACL prava
    - Windows
    - built-in sifrovani
    - az 4TB

- VFS (Virtual File System)
    - abstrakce file system (rozhrani)
    - navrzene UNIXem
    - file systemy musi implementovat toto rozhrani (NTFS, FAT, UDS, AFT)
    - obsahuje ukazatale na fukce (C obdoba rozhrani)
    - hlavni komponenty VFS
        - superblock (aktualne namountovany disk)
        - inode (konkretni soubor/slozku)
        - file (aktualne otevreny soubor)
        - dentry - informace o directory

- Proces a soubory
    - kazdy proces ma svuj pracovni adresar
        - ulozeno ve fs_struct v PCB
    - fs_struct dri otevrene soubory daneho procesu
        - open() vraci index do pole - file handler
        - pcb->files->fd[0] = stdin
        - pcb->files->fd[1] = stdout
        - pcb->files->fd[2] = stderr
    - po zavolani close() a znovu zavolani open() se vrati prvni prazdny index (fd)
        - moznost takhle presmerovat stdio
            - close(fd[0]); open('my-file') -> dostane index 0

- Installable File System (IFS)
    - Windows (obdoba VFS na UNIXu)
    - 3 modely operaci
        - neni pozadavek na to je implementovat vsechny
        - file system - ovlada souborovy system na disku (image)
        - Mini Filter - rozhrani napr antivirus a indexovani sluzeb
        - FS FilterDriver - modifikuje existujici souborovy system
    - pouziva IO Request Packety pro komunikaci

- IO Request Packet (IRP)
    - struktura pouzita pro komunikaci mezi ovladacem zarizeni a kernelem
    - popisuje pozadadovane operace
    - ukladaji se do fronty
        - OS je muze preusporadat
    - Vetsinou jsou vytvorene IO Managerem jako odpoved volani IO OS
        - API od uzivatelskeho procesu
    - Nemusi byt limitovane pouze na souborovyc system
        - napriklad setreni energiie - vypinani disku nebo monitoru (po tom co je neaktivni po nejakou dobru - neprijak zadny IRP)
    1) Usermode zavola API OS (pomoci RTL)
    2) v Kernelu (CPL0) IO Manager prijme pozadavej a vytvori IRP ktery posle danemu zarizeni (napr disku)
    3) tenhle pozadavek zachyti Filter Manager a prozene jej prislusnym filtrem (napr antivirus)
    4) Az pote driver udela prislusne operace nad diskem
    3) Az zarizeni odpovi, posle vysedek pozadavku danemu procesu v userspace (napr pres EAX)

- Filter Manager
    - aktivovan pri nacitani mini filtru
    - Pripojeni na zasobnik file systemu konkretniho disku
        - registruje se na specificke IO operace - krere zachyti
    - filtry jsou pripojene v poradi danem jejich prioritama
        - napr anivirus ma vetsi prioritu nez idex sluzby?
    
- Fast IO
    - IRP je nastroj pro vsechno
        - synchroni i asynchroni presuny dat
        - data pro presun z/do cache, page faulty, ...
    - Fast IO se snazi zrychlist SYNCHRONI operace nad daty krere jsou pritomny v cachi daneho disku
        - v takovem pripade je presun data process-to-process primo pres systemovou cache
        - vynecha file system a HW stack - neco jako loopback na localhost
    - Pokud Fast IO selze, provede IRP (klasika)

- IO subsystem (obecny princip)
    1) Po volani OS API, OS vytvori IO pozadavek a preposle ho IO sybsystemu
    2) Valajici vlakno je zablokovano dokud se pozadavek nedokonci
    3) IO se da do fronty dokud si ho driver nevezme
    4) Driver but preposle pozadavek jinemu driveru nebo ho sam vykona
        - nejaky HW vykona pozadavek hned (sitovka, zvukovka)
        - nejaky si ho jen da do fronty a pres IRQ (disk, GPU) oznami ze ho prijal
    5) Po dokonceni pozadavku je vlakno opet probuzeno a muze se planovat
        - nemusi hned -> co kdyz je vlakno blokovano jeste nad necim jinym (napr debugger) viz DELOS

- NE2000 (NIC)
    - jednoducha referenci sitova karta
    - jeji jednoduchost -> stala se popularni
    - odesilani dat
        - sekvence prikazu odelanych na port
        - vzdy cekame dokud se nevycisti prislusne bity (flag - muzem posilat dalsi prikaz)
    - prijimani dat
        - notifikace pres IRQ
        - interrupt handler precte data

- Handling IRQ
    - IRQ - signal generovany HW je poslan do CPU
    - udalost oznamujici OS ze se ma dany HW obslouzit (neco se stalo - udalost)
        - zmacknuta klavesa, prisly packety, tik casovace, deleni 0, etc.
    - IRQ je premapovano na index do IDT
    - Jadro CPU prestane vykonavat aktualne vykonavany program a zacne vykonavat obsluhu preruseni (dano podle IRQ)

- PIC
    - Programmable Interrupt Controller
    - Obsluhuje 8 IRQ -> jsou pouzity 2 PIC cipy (kaskadove zapojeni)
        - master
        - slave

- IRQ Konflikt/sdileni
    - co by se stalo kdyby 2 zarizeni sdilely stejny IRQ?
        - pokud nejsou zarizeni pouzivana ve stejny cas - OK (napr v DOSu hrani her - zvukovka a napr tiskarna nebudou pouzivany zarovek - stesti)
        - nevyhoda je ze cely system muze zamrznout pokud neobslouzime preruseni spravne 
            - viz DELOS (posilani ACK to PIC)
    - Systematicke reseni
        - Message-Signaled Interrupts 
            - zarizeni ocekavani sdileni PIC line/cislo IRQ
        - Zarizeni ma navic ID ktere posle PICu aby bylo mozne rozlisit o jake zarizeni se jedna
            - muzeme rozlistit vice zarizeni pomoci jednoho IRQ -> Advanced PIC

- Advanced PIC
    - Umi preposilat mnohem vice IRQ nez obycejny PIC
    - Dovoluje integraci se SMP
        - kazde jadro CPU ma vlastni APIC spolu s lokalni tabulkou pro premapovani udalosti na IDT cisla
            - vyzaduje Message-Signaled Interrupts
            - take se jim rikal virtualni IRQ
        - V Systemu je pak hlavni IO APIC, ktery rozesila pozadavky mezi jednotliva jadra CPU
            - APIC bus

- Top Half
    - IRQ musi byt obslouzeno co nejrychleji jinak muze cely PC zamrznout (stejne jako pri IRQ konfliktu) - dulezite timeouty
    - handler dela pouze nutne minimum
        - co zbyva dodelat je zarazeno do fronty
        pro pozdejsi zpracovani
        - hanler oznami (ack) konec obsluhy preruseni
    - Praci kterou vykona interrupt handler = TOP HALF!!

- Bottom Half
    - V kernelu jsou demoni (vlakna/procesy/servery) ktere periodicky vykonavaji odlozene tasky (nasledky konceptu Top Half)
    - Vykonavani techto odlozenych cinnosti = Bottom Half
        - napriklad proces neprijme UDP datagram hned potom co byl prijat (od sitovky) ale az co nejaky bottom-half demon zpracuje odlozenou praci
    - dva druhy bottom-half handleru
        - kriticke - aka SoftIRQ, v Linuxu
        - Naplanovane - aka Taskled a exekutny SolfIRQkem, v Linuxu

- SoftIRQ vs Tasklet
    - SoftIRQ
        - staticka alokace na konkretnim jadre
        - obsluhuje time-critical events (na Linuxu treba sit)
        - SoftIRQ stejneho typu se mohou vykonavat soucasne na ruznych jadrech
        - jeden ze SoftIRQ executuje tasklets
    - Tasklet
        - staticka i daynamicka alokace
        - tasklety RUZNEHO typu se mohou vykonavat soucasne
            - poze jeden tasklet stejneho typu se muze vyhonat v jeden okamzik -> zjednoduseni uzamikani -> zlepsuje propustnost
        - musi byt dokoncen atomicky (nemuzeme ho suspendovat)
        - vykonava se na stejnem jadre jake ho naplanovalo
        
- Linux: Workqueue
    - kernel se snazi vykonat odlozenou cinnost (SoftIRQs a Tasklets) ASAP
        - napriklad po dokonceni posledniho ISR
        - konkretne kdy zalezi na konkretni architekture OS a kernelu
    - Workqueue je dalsi type odlozene cinnosti - nemusi byt hotovat ASAP
        - rikame kernelu ze tato cinnost se ma udelat nekdy v budoucnu
    -  kazda workqueue muse mit svoje vlakno na danem jadre v SMP
        - protoze nekazdy ovladac potrebuje workquque, system poskytuje jednu sdilenou

- Polling
    - ISR vzdy obsluhuje prislusne IRQ (casovac, deleni 0, page fault, klavesnice, ...)
    - Polling muze obsuhovat dalsi zarizeni
        - napriklad ovladac disku v IO subsystemu nemusi cekat na IRQ aby se dozvedel o vysledku (konci) operace
        - periodicky se pta (poll) resp. cte stavovy bit
        - -> tohle ale neni duvod kompletne opustit princip IRQ!
        - pri obsuze velkeho mnozstvi IO pozadavni, polling resi docela dost podstatnych problemu

- IRQ->ISR vs Polling
    - pri pollingu, ovladac ceka ve smycce dokud neni prace hotova
    - s velkym poctem IRQ, procesor musi "vydrzet" odpovidajici pocet kontext switchu (CPL)
        - napr pri statisicich IRQ za sekundu -> problem
        - s kratkymi cekanimi, IO vlakno se pozastavi, jine vlakno se naplanuje a hodi na CPU, tim padem se vymaze CPU cache -> zpomalovani IO vlaken -> musime znovu nacist data do cache aby byl vokon takovy jaky byl

- IO_uring
    - Linux (rozsireny Barkeley Packet Filter aka eBPF)
    - redukuje prepinani user-kernel -> zvyseni vykonu
    - program hazi IO pozadavky do fronty
        - OS zpracuje userspace frontu kterou kernel muze precist v jednom prepnuti kontexu (kdyz se preplanovava)
    - Kenrel zpravuje IO pozadavky a vytvori seznam odpovedi -> prgram si je pote zpracuje

- DMA - motivace
    - Direct Memory Access
    - Cteni dat pomoci portu a IRQ nemusi byt tak efektivni (pokud data nejsou dostatecne mala)
        - napr 9600 boud modem trigruje IRQ priblizne kazde 2ms (pri penosu jednoho znaku za milisekundu)
        - dokonce i floppy drive presouvat prilis mnoho dat
            - co potom moderni disky a sitovky?
        - Potrebujeme neco na kopirovani dat mezi zarizenimi a systemovou pameti (RAM) - DMA
    
- DMA - pouziti
    - omezeny pocet DMA kanalu
        - zadny kanal nemuze byt sdileny
        - ovladac musi pockat dokud DMA kanal nebude volny (k dispozici)
        - ne vsechny kanaly jsou dostupne pro vsechny zarizeni (HW kompatibilat)
        - kernel si drzi seznam DMA kanalu spolecne s jejich stavem (available, ...)
    - jadro pouze zahaji (nastavi) prenos a jde si delat neco jineho
        - IRQ signalizuje konec prenosu dat
        - Legacy HW (floppy drive, LPT, lrDA, ...) pouzivaji 16-bit adresy
            - Limituje DMA poze na prvni 16 MB RAM
        - DMA s PCI sbernici pouziva defaultni 32 bitove adresovani
            - 64 bitove adresovani v double-access cycle mapping mode