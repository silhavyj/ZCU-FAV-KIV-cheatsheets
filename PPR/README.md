## 01-topology

- Jake jsou nevyhody hybridniho CPU?
- Co je Intel Quick Path?
- Co je NUMA a jake jsou jeji vyhody a nevyhody?
- Rozdil mezi SMP a ASMP?
- Vyhody a nevyhody ASMP?
- Princip multiprocesoru s distribuovanou pameti
- Proc se u multiprocesoru s distribuovanou pameti pouziva simplexni komunikace?
- Na cem zavisi vykon pri pouziti distribuovane architektury?
- Jaka je vyhoda pri zapojeni do toriodu proti zapojeni do mrizky?
- Pipis algoritmu distribuovaneho souctu
- K cemu se pouziva a jak je rizeno Systolic Array
- Princip simplexni komunikace
- Co je to Front-Side Bus?
- Pro jaky typ uloh se hodi multiprocesor s distribuovanou pameti?
- Priklad nepravidelne sitove topologie

## 02-temporal

- Princip temporalni logiky
- Co plati o vystupu systemu, ktery je hladny watchdogem?
- Proc hledame invarianci pri dokazovani spravnosti programu pomoci temporalni logiky?
- Co je to invariance?
- Popis problemu vecericich filozofu z pohledu temporalni logiky
- Nevyhoda protokolu Aloha z pohledu temporalni logiky
- Princip Watchdoga a jaka je jeho hlavni vlastnost
- Jak se da detekovat, ze program pocita spatne vysledky + priklad
- Jakym zpusobem paralelne detekovat chyby HW i SW?
- Princip master a shadow kodu
- Co je to data audit?
- Jak se da predejit nehodam za runtimu?
- Rozdil mezi fail-safe a fault-tolerant (+ priklad)
- Rozdil mezi active a passive safeground?
- Princip inkrementalniho rizeni insulinove pumpy
- Obecny byzansky problem
- Jaky je predpoklad reseni obecneho byzanskeho problemu?
- Proc musi byt komunakce v obecnem byzanskem problemu synchronizovana (+ jak se resi synchronizace)?
- Co je fail-stop a fault tolerant v obecnem byzanskem problemu?

## 03-gpgpu

- Po jak velkych blocich nacita CPU data z pameti?
- K cemu slozi `__attribute__((packed))`?
- Cim je rizeno GPGPU?
- Jakym mechanismem se prenasi data z adresniho prostoru procesu na grafiku?
- Jaky je pozadavek na data, ktera se maji zpracovat na grafice?
- Task vs Data paralelism (co je na co jak optimalizovane)?
- Princip SIMT
- Kolikrat se zpomaly nacitani dat z pameti, kdyz nebudou promenne/struktury zarovnany na velikost strojoveho slova?
- Co je to strojove slovo?
- Co musi byt splneno aby slo pouzit prednacitani dat z pameti?
- Jaka je nevyhoda spare memory access?
- Jaky datovy typ ma navratova hodnota z OpenCL kernelu?
- Co je to ND-range?
- Co znamena `0` v `get_global_id(0)`?
- Na co se mapuje jeden work item?
- Jaky je vztah mezi work size a work group size?
- Jaky je vztah mezi work size poctu work itemu?
- Co je to local size?
- Co je to wavefront a jak se da ridit?
- Typy pameti na OpenCL zarizeni
- Na cem zavisi vykon pri pouziti globalni pameti na OpenCL zarizeni?
- Co je to bank-conflict?
- Jak je definovana velikost privatni pameti OpenCL zarizeni a co se deje pri preteceni?
- Popsat SPIR-V
- Princip Incoherent Branching (nejlepsi a nejhorsi pripad)
- Jak wavefronta ovlivnuje pouziti bariery na GPGPU?
- Rozdil mezi Fence a Barrier
- Co je CPU fallback?
- Co dela klicove slovo `restrict` a klicove slovo `volatile`, priklad pouziti?
- Co je to online kompilace OpenCL programu?
- Jaka je podminka pouziti struktury na OpenCL (jako parameter kernelu)?
- Jak lze synchronizovat work-groupy?

## 04-sycl

- Co je to SYCL?
- Co je to AMP?
- Rozdil mezi explicitnim a implicitnim pristupem pri predavani bufferu u SYCL
- Kde a jak se pouziva execution policy?
- Princip cache-cooling efektu
- Co jsou to hot in cache data?
- Jak se TBB snazi snizit cache-cooling efekt?
- Jak funguje task-stealing scheduler?
- Co predstavuji listy a interni uzly pri rozkladu tasku, jake data jsou nejvice cache-hot?
- Je `parallel_for` turingovsky kompletni?
- V cem se lisi `tbb::parallel_for` a `tbb::parallel_reduce`?
- Jak se agreguji mezivysledky pri pouziti `tbb::parallel_reduce`?
- Co je to split konstruktor?
- K cemu se da pouzit `prallel_deterministic_reduce`?
- V cem se lisi `tbb::parallel_do` od `tbb::parallel_for` a `tbb::parallel_reduce`?
- K cemu slouzi `tbb::parallel_do_feeder`?
- Typy filtru u `tbb::parallel_pipeline`
- Proc se vytvari nekolik instance jedne `tbb::parallel_pipeline`?
- Princip Flow grafu
- Cim se spusti vypocet na prvnim uzlu ve flow grafu?
- Load-Balancing na CPU a GPU
- Proc nemuze byt funckti operator `const` u `tbb::parallel_reduce`?
- Vyhoda TBB oproti OMP

## 05-pipeline

- Roste urychleni programu linearni s poctem jader?
- V cem spociva efektivita sablon v C++?
- Princip Out-of-Order Execution
- Co jsou to nepojmenovane registry?
- Co to jsou mikroinstrukce a jake jsou faze pipeliny
- K cemu vede paralelizace pipeliny?
- Cim je dany pocet fazi pipeliny?
- V cem skociva problem podminenych skoku + 2 moznosti reseni?
- Co je to rollback v pipeline?
- Co obsahuje BTB?
- Co to je saturing counter?
- Kolikrat dojde k miss predikci `{ 1, 2, -5, -4, 2, 1 }` (3 bitovy saturing counter)
- Jaka je vychozi hodnota saturing counteru?
- Princip dvouurovnove adaptace pri branch predikci
- Na kolik instrukci se prevede podminka `if`
- Priklad eliminace podminek + jake to muze mit vyhody a nevyhody
- Na jakou instrukci se v C++ preklada ternarni operator
- V cem spociva pametova a datova zavislost?
- Jak se da efektivne nastavit hodnota na 0?
- Jak muze zpusobit ze bude paralelni program pomalejsi nez seriovy?
- Jaky je rozdil mezi `constexpr` a `consteval`?
- Co to je LLC miss?
- Co to je instruction starvation?
- Jaka je nevyhoda a nevyhoda velkeho poctu fazi v pipeline?
- Kdy je dobre pouzit `cmov`?
- Co je to falesna zavislost?
- Co to je CPI?
- Rozdil mezi instruction starvation a mispredikce vetve

## 06-simd

- Na cem zavisi urychleni pri pouziti SIMD instrukci?
- Co umoznuje vetsi urychleni? Vektorizace nebo paralelizace?
- Automticka vs manualni vektorizace
- Proc musi komilator znat pocet opakovani cyklu aby provedl auto-vektorizaci?
- Cemu muze pomoct loop-unrolling?
- Co implikuje to ze ma smycka vice vystupnich bodu a jake to ma nasledky na vektorizaci?
- Princip maskovani a podmineneho nacitani + vyhody / nevyhody
- AoS vs SoA
- Princip Scattered Load and Shuffling (AoS)
- K cemu slouzi Expression Templates? Cemu zabranuji? V jake funkci/metode je implementovana samotna logika?
- Co je Fused Multiply Add?
- Jaka je vyhoda a nevyhoda pouziti jednoho zamku pri psitupu k datove strukture? Jak se tento problem da resit SW a HW?
- Co je to fallback path?
- Co se deje pote co jadro skoci zpet na fallback path?
- Co je to `ZF` a kde se nachazi?
- Proc se u HLE i RTM spinlocku nastavuje eax = 0?
- Kde a k cemu se pouziva xbegin?
- Jak souvisi loop unrolling a branch prediction?
- Ja se da implementovat `min` bez pouziti `if`?
- Co obahuje `edx` pri pouziti HLE a RTM?

## 07-os

- K cemu s pouziva Ada, z ceho vychazi a kde vznikla?
- Princip Rendez Vous?
- Jak funguje task v Ade?
- Co se stane kdyz nikdo nezavola vstupni entry u tasku v Ade?
- K cemu slouzi select v Ade?
- K cemu v Ade slouzi klicove slovo `type`
- Jaky je rozdil mezi funkci, procedurou a vsupnim volanim (entry) u Ada objektu?
- Rozdil mezi taskem a typem v Ade
- Jaky mechanismus se pouziva v Ade v pripade ze podminka vsupniho volani neni splnena?
- Proc nejde rades-vous implementovat v Jave jen pomoci monitoru a v C/C++ ano?
- Co je to Job objekt?
- Co je planovaci jednotka Windows a Linuxu?
- V cem spociva problem inverze priorit v real-time OS?
- Co je impersonating?
- Princip TLS
- Co dela klicove slovo `__declspec`
- K cemu se pouziva Win API ThreadPool?
- Jaka je vyhoda pouziti Fiber thread a jak se vytvari?
- Co to je FLS?
- Co je to lazy init + priklad?
- Muze fiber pristupovat k TLS?
- Jak se nastavuje priorita fiberu?
- Co je to UMS a jak se vytvari a jak se lisi od fiber?
- Zpusoby Win API synchronizace (alespon 6)
- Co to je APC?
- K cemu slouzi pojmenovana podminkova promenna?
- Co je to SimLock, jake jsou vyhody jeho pouziti a proc se neda pouzit v rekurzi?
- Jaka je podminka atomicity operace?
- Kde se pouziva mechanismus posilani zprav? Jaky je rozdil mezi PostMessage a SendMessage?
- V cem je komplikovanejsi synchronizace procesu oproti synchronizaci vlaken?
- K cemu se pouziva CreateFileMapping?
- Co to je .bss?
- Jak je zajisteno korektni ukonceni vlakne pomoci `return NULL`?
- Co je to POSIX a co obsahuje?
- S cim pracuje afinita? S vlakem nebo s procesem?
- Jaky je rozdil mezi globalnimi, lokalnimi a daty na zasabniku z pohledu WinAPI vlaken?
- Co je ulozeno v kontextu vlakna?
- Jak se lisi planovani Fiber a UMS?
- Jaky vliv ma APC na planovani vlaken?
- Jak se da pouzit podminkova promenna pro synchronizaci procesu?
- Co to je SRW lock? Kdy je vhodne jej pouzit?
- Princip TimerQueue
- K cemu slouzi `WM_COPYDATA`?
- Jak OS zajisti ze maji 2 procesy pristup ke stejnemu DLL?
- Jaky je rozdil mezi OOP objektem a POSIX objektem?

## 08-prefix

- Co je problem usporadane promenne?
- V cem spociva naivni pristup paralelniho pocitani prefixovych sum? Jake to ma nevyhody? Na cem zavisi efektivita?
- Princip optimalniho algoritmu prefix-sum
- Jaky je rozdil mezi inkluzivnim a exkluzivnim pristupem pri vypoctu prefix-sum?
- Jaka je aplikace algoritmu prefix-sum?
- Jaky je princip segmentovaneho scanu pri pocitani prefix-sum?
- Jaka je casova slozitost pri oplimalnim prefix-sum?
- Co je to Line of Sight?

## 09-evolution

- Moznosti kvantifikace vhodnosti reseni u evolucnich algoritmu
- Vyhody/nevyhody evolucnich algoritmu
- Obecny popis (kroky) evolucniho algoritmu
- Co vyjadruje fitness funkce?
- Jak se lisi problem predcasne konvergence a stagnace?
- Co by mela (teoreticky) splnovat pocatecni populace?
- V cem spociva naivni myslenka pri tvorbe potomku?
- Jak je definovana relativni chyba?
- Jak nebo cim se zaruci minimalni presnost u EA?
- Cim je davan velikost pocatecni populace?
- Co je to geneticky drift?
- Cim je ovlivneny geneticky agoritmus?
- Co ovlinuje `F` a `CR` parametr? Kdy se tyto parametry nastavuji?
- Strategie mutaci
- Co je to meta diferencialni evoluce?
- Co vytvari mutace u evolucnich algoritmu?
- Co je to chromozom?
- Co je to rekombinace? A jake jsou mozne strategie?
- Jak se da prokazat konvergence a co je to zastavovaci podminka u EA? Jake tam je uskali?
- Rozdil mezi jemne clenenou a hrube clenou distribuovanou evoluci
- Co je to tzv island model u EA? Jak se v nem resi problem stagnace?
- Co je to tzv souostrovni model u EA a jak se lisi od island modelu?
- Jak se lisi dominalne a slabe lepsi potomek? V cem se tento termin pouziva?
- Co to je paretova fronta a paterova optimalni mnozina?
- Je GA podmnozinou EA?
- Jak se lisi GA od EA?
- Co je to vice ucelova optimalizace + priklad
- Jake je nevyhoda toho kdyz je pocatecni populace prilis velka?
- Jak spolu souvisi dimenzita problemu a velikost pocatecni populace?
- Jaky je rozdil mezi krizenim a rekombinaci?

## 11-distributed

- Podle jakeho design patternu funguje SETI@Home?
- Je u SETI@Home pozadavek na stejny HW?
- Co to je MPI? Je u MPI pozadavek na stejny HW?
- Co to je Single-Purpose Computer?
- Princip Wormhole Switching, co je v tele prvniho a posledniho packetu?
- Kdy musi byt distribuce prace dynamicka a kdy na pevno?
- Jaka je nevyhoda Store & Forward pro routovani?
- Jak funguje adaptive switching?
- Co je to SPMD?
- Rozdil mezi homogeninm a heterogenim distribuovanem prostredi
- Jake urychleni prinasi Farmer-Worker, co je treba zanedbat?
- Jaka je nevyhoda Lampartovo hodin?
- Princip lampartovo hodin (co to je, jak to funguje, co se deje pri odesilani/prijimani zpravy)
- Vysvetli: pokud je C(a) < C(b) nemuzeme dedukovat ze a zposobilo udalost b
- K cemu slozi vektorove hodiny? Jak se lisi od Lampartovo hodin?
- Co musi platit aby: pro udalosti a a b plati ze VC(a) < VC(b)?
- Co je to virtualni topologie a jak v ni funguje komunikace mezi procesy?
- Jak se daji odhalit klastry ve virtualni topologii?
- Co je to "Kilometer Square Array"?
- Vysvetleni funkci map a reduce v distirbuovanem prostredi a v SMP
- Jak se daji napevno namapovat procesy na jednotlive jadra SMP?
- Jake jsou hlavni vlastnosti distribuovaneho systemu?