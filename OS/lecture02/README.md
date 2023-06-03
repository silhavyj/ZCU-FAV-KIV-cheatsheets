- MMU
    - CPU -- virtual --> MMU -- physical --> pamet
     - kus HW ktery preklada adresy
        - SW reseni by bylo pomale
     - segment:offset
        - offset je relativni vuci segmentu
        - virtualni adresa se prelozi na linearni (pokud neni strankovani jedna se primo o fyzickou)

- Protected mode
    - ochrana prepsani kodu kernelu z userspace (DPL)
    - vyber segment selector -> z neho pak segment deskriptor
    - prepnuti do protected mode
        - zakaz preruseni
        - nastaveni bitu v cr0
        - lgdt
        - lldt
        - nastav ss
        - perform long jump to fix cs segment
        - ret

- deskriptor segmentu
    - bazova adresa
    - limit
    - typ
        - code
        - data
    - access control flagy
    - dodatecne flagy

- izolace kernelu
    - kernel musi vytvorit tabulky deskriptoru
        - GDT (globalni pro kernel, GDTR registr)
        - LDT lokalni pro proces
    - lgdt i lldg vyzacuji CPL0 -> proces si ji nemuze sam zmenit

- segment selektory
    - 6 registru
    - CS, DS, SS, ES, FS, GS
    - adresa je prefixnuta segment selektorem
    - CS:funkce
    - DS:adresa dat
    - SS:ebp-offset_funkceargument
    - EBP drzi ESP pri volani funkce

- Segmentace
    - vyhody
        - ddeleni kerneu a procesu
        - sdileni segmentu
            - read-only segments -> shared lib
    - nevyhody
        - jak nastavit velikost segmentu?
        - jak resit fragmentaci?

- Strankovani
    - eliminuje nedostatky segmentace
    - x86 dovoluje kombinaci obou
    - vyzaduje 32 nebo 64 rezim CPU
    - rozdeleni pameti na bloky o velikosti 4kB, 2MB, 4MB, 1GB
    - velikost frame = velikost stranky

- Tabulka stranek
    - adresa ulozena v cr3
    - zmena vyzaduje CPL0
    - zmena cr3 vzdy pri prepnuti kontextu
    - ma priznaky:
        - kernel/user
        - writable
        - present (je v RAM nebo ve SWAP)
            - pokud ve SWAP tak adresa = adresa ve SWAPu ne v RAM
        - dirty, accessed
            - RR, LRU (algoritmy pro vymenu stranek)

- Hirearchicke strankovani
    - Dovoluje az 16PT ale neni vetsinou (skoro nikdy) treba
    - Je rozume vybalancovat zanoreni (levely) a velikosti stranky
    - Cim mensi velikost stranky -> ti vic levelu potrebujeme -> vetsi rezije!
    - Cim vetsi velikost stranky -> tim mene levelu potrebujeme
    - pro frame = 4kB -> virtual addr 32 bitu
        - 10 bitu - page table
        - 10 bitu - page
        - 12 bitu offset
    - pro 4MB -> virtual addr 32 bit
        - 10 bitu page table
        - 22 bitu offset

- TLB
    - po vypoctu fyzicke addr ulozit do cache
    - zrychleni celeho vypoctu -> nemusime pocitat 2x
    - pri prepnuti kontextu je nutny flush (addressy nejsou platne v nasem aktulnim adresnim prostoru)

- Shared memory
    - sdileni dat mezi procesy
    - na stejnou adresu ve dvou (nebo vice) adresnich prostorach namapujeme stejny ramec
    - o konzistenci se staraji procesy sami kernel "to pouze napoji"

- Sdileny kod
    - kdyz jdou sdilet data proc ne i kod?
    - nastavime pages jako read-only
    - nactu dll (so) do adresniho prostoru obou procesu

- Copy on Write
    - Pokud 2 procesy sdili pamet
    - stranky nastavene jako read-only,
    - pri pokusu o zapis CPU vyhodi vyjimku
    - vime kdo a kam se snazil zapsat
    - udelame kopii stranky na novej ramec a premapujeme ho v pameti daneho procesu
    - proces pokracuje dal jakoby se nic nestalo (ma ted kopii originalnich dat, neovlivnil ostatni procesy)

- SWAP
    - na disk, USB flash, sit, atd.
    - odkladani stranek pomoci algoritmu RR, LRU, ...
    - tyto algoritmy vyuzivani access a dirty bity
    - predstirani ze mame vice adresniho prostoru nez ve skutecnosti mame
    - vyrazne snizi vykonnost!!
    - presun stranek zajistuje DMA
    - pro page fault
        - kouknem jestli je stranka ve swapu
            - namapujem (RR, LRU, ...)
        - kouknem jestli volnej ramec (kdyz nebyla na mapovana)

- Page Fault
    - vlakno se zablokuje
    - kernel neresi page fault hned (velka rezije) ale ceka bud dokud nenastane n page faultu nebo do timeoutu

- File Mapping
    - namapujeme soubor do ramky (po blocich o velikosti jedne stranky)
    - pri zavirani ulozime pouze ty stranky ktere maji dirty bit = 1
    - minimalizaci IO operaci

- PAE
    - predchudce 64 bit rezuimu CPU
    - rozsireni adres o 3 bity -> 36 bitova adresa
    - PAE se zapina extra