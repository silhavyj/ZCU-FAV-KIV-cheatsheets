- Protected mode je HW izolace procesu a kernelu

- CPU startuje do realneho rezimu
  - Nema izolaci!!

- Registry
    - General Purpose (A B C D)
    - Segmentovy (rozdeleni pameti na segmenty)
        - CS (kodovy segment)
        - DS (datovy segment)
        - SS (stack segment)
        - ES (extra segment)

    - Indexove registry
        - SI (source)
        - DI (destination)
        - BP (drzi hodnoty SP kdyz se zavola funkce - base pointer)
        - SP (stack pointer)
     
     - Flags
        - state registers
        - vysledek operace
        - carry bit, parita, atp.
        - zakaz preruseni IF

     - ridici registry
        - CR0 (povoleni strankovani, rezim CPU, ...)
    
     - ostatni
        - CR2 (adresa page faultu)
        - CR3 (adresa page dir)
        - IDTR (adres tabulky vektoru preruseni)
        - atd

- Real mode addressing
    - segment:index (adresa ma 20 bitu)
    - segment i index jsou oba 16 bitu
    - add = (segment << 4) + index
    - => adresace mas 1MB

- Memory map
    |  Addr | Entity  | Desc  |
    |---|---|---|
    | 0  | IDT  | 256 x 4B  |
    | 0040  | ROM BIOS  |  promenne  |
    | A000  | EGA  | grafika (pixely)  |
    | B000 |  MDA  | cernobily text |
    | B800 |  CGA  | text mode |

- MS DOS
    - soborovy system + konzole
        - zbytek pomoci driveru
    - Bootstrap na 80286
        1) Po zapnuti v real mode
            - Full access pro 1MB ramky
        2) CS:IP nastaven na F000:FFF0
            - skoci do kodu BIOSu
        3) BIOS zacne hledat disky
        4) Po nalezeni disku nacte prvni sektor 0x7C00 (512B)
            - nacte MBR
        5) z MBR nacte OS loader
            - Boot flag 0x00 (nebootovatelne), 0x80 (bootovatelne)
        6) Muze byt rovnou os-kernel loader nebo OS-selector (GRUB)
        7) Jamkmile MBR loader najde partition nacte OS-kernel do pametu
            - Specificky IO.sys a MSDOS.sys
            - jsou ulozeny hned za sebou v 1. sektoru
            - loader je moc malej na to aby implementoval jakykoliv FS
        8) IO.sys se po nacteni executne
            - rozhrani pro IO sybsystemy
            - Elementarni HW interface
        9) IO.sys preda kontrolu MSDOS.sys
            - OS kernel API nad HW
        10) Pokud existuje CONFIG.sys
            - Nacteni dalsich driveru - mys, CDROM, zvukovka, etc.
            - XMS EMS (extended memory -> adresace az 4GB)
        11) MSDOS.sys nacte a provede prikaz
            - Interpretr (aka shell) COMMAND.com systemovy soubor
        12) Command.com nacte jeste autoexe.bat
            - defakto je po-spusteni .bat
            - bat = obsahuje davkove prikazy
        13) C:\ DONE


- Windows 9X Bootstrap
    - Win.com nacetr 16-bit Windows
        - Win/386 loader prepnul CPU do protected modu
        - Dovoli mem access vyssi nez 1MB

    - S Windows 9X, IO.sys nastavil veci v real-mode, spustil wun.com az autoexec.bat dokoncil praci
        - drivery v protected modu byly porblem
        - pouziva protected mode pro process/kernel izolaci
        - Real-mode drivery potrebuji virtualizaci V86 pro exekuci (rozdilny mody CPU, rozdilna sirka registru, ...)

- Kernel vetsi nez 1MB
    - CPU zactne v real-mode s 20 bitovyma adresame (az 1MB adresave)
    - prepnuti z protected modu do real-modu zachovava limit pameti (segment descriptory) jako pro protected mode
    - Muzeme adresovat az 4GB

- Unreal mode
    - neni oficialne dokumentovany
    - spousteni programu v 32 bitovem modu vyzaduje nastaveni GDT a LDT (tabulky deskriptoru)
        - Rozdeli pamet na segmenty
    - pri prepnuti z real do protected nastavime limity na max
        - bez strankovani
    - Pak se prepneme zpatky do real-modu (unreal mode)
        - Muzeme pouzit 32 bitovy index registry pro adresaci
        - 4GB flat address space

- Pokud program potrebuje vic pameti nez je k dispozici
    - XMS a EMS - memory managers, prepnou CPU do 32 bit mode a zustanou tam
    - Poskytuji API programkum v real-modu pro kopirovani 64KB bloku z/do pameti nad 1MB

- Overlays
    - Uchovava co nejmene kodu v pameti
    - Pouze cesta z listu do main
    -v pameti jsou pouze aktivni moduly (reps funkce)
        - setreni pameti

- UEFI
    - Unified Extensible Firmware Interface
    - Nastupce BIOSu
    - Vyuziva GPT (nastupe MBR)
    - MBR ma limit 2TB, GPT ne
    - EFI binarky pro load OS
        - nahrazuje treba boot loader, vic safe
        - hleda v GPT

- MSDOS API
    - Interrupty 0x21
        - IDT na 0x0
        - Do AX cislo sluzby (viz dokumentace daneho API)

- Obsluha API
    - Napl registry dle specifikace API
    - Zavola se int 0x21
    - SS:SP na stack a CPU ulozi flags a CS:IP
    - kernel prevezme kontrolu nad CPU
        - precte/zapise do registru (AX navratova hodnota)
    - Zavola se IRET a pokracujeme na CS:IP na vrcholu stacku

- Text mode video memory
    - 2B na znak (1. byte - dany znak, 2. byte pozaci a popredi (4 + 4 bity))