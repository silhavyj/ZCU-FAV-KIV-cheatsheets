1. Jak se lisi spousteni noveho procesu UNIX vs Windows?
2. Jake jsou navratove hodnoty forku?
3. Vyhody/nevyhody vforku? 
4. Jak funguje clone
5. Co to je pthread_atfork?
6. Jak funguje preruseni?
7. Co obsahuje PCB?
8. Jake jsou stavy procesu z pohledu SMP?
9. Jak je implementovan resp reprezentovan stav procesu v jadre z pohledu datovych struktur?
10. Rozdil mezi zombie a sirotkem? Muze se ze zombie stat sirotek?
11. Proces zbavovani zombie?
12. Jaka ja navratova hodnota exec?
13. Jak mezi sebou souvisi fork, pthread_create a clone?
14. V cem vsem se lidi child a rodic? Co maji jineho? Co maji stejneho?
15. Typy semaforu a jak funguji?
16. Jake ma vyuziti semafor na uniprocesoru?
17. Jake jsou pristupy pri probouzeni vlaken spici nad semaforem?
18. Jak funguje producent konzument - kod?
19. Jak se lisi mutex a binarni semafor?
20. Jak funguje pipe a co je to pojmenovana pipe?
21. Jake existuji zpusoby synchronizace vlaknu/procesu?
22. Jak se lisi PostMessage a SendMessage?
23. K cemu slozi kod zpravy WM_COPYDATA?
24. Jake signaly nejdou z pohledu procesu ignorovat a jak se lisi?
25. Rozdil mezi Process-Directed Signals a Thread-Directed Signal
26. Jak jsou nastavene handlery signalu po pouziti fork + exec?
27. Jake znas registry + typy?
28. Co je to segmentace?
29. Adresovani v realnem modu
30. Memory map v realnem modu
31. MS DOS bootstrap
32. Jak se zavadi kernel > 1MB?
33. prink cemu slouzi XMS a EMS?
34. Co jsou to Overlays?
35. Rozdil mezi Legacy bootem a UEFI?
36. Co je to MMU a k cemu slouzi?
37. Jak se presouvaji stranky z RAM na disk?
38. Jak funguje prepnuti do protected modu (step by step)? 
39. Hierarchicke strankovani?
40. Jak funguje copy on write?
41. Jak fungujCoe reseni page faultu? Resi se page fault hned jak nastane?
42. Pametove mapovane soubory
43. PAE
44. Mikro kernel
45. Jake existuji typy ringu + priklady v jakem co typycky bezi
46. Resolvovani adres u dynamickych knihoven
47. K cemu slouzi TSS?
48. Typy vyjimek?
49. Co je to VESA?
50. Zretezene ISR
51. Cooperativni planovani
52. Jak je implementovana afinita a co to je?
53. Jak funguje RR s prioritama? Muze vzbiknout deadlock?
54. Completely Fair Scheduler
55. Staticka vs dynamicka priorita
56. SMP Bootstrap x86
57. Co je to a k cemu je crt0?
58. Jak vypada pametovy prostor procesu?
59. Blokova vs znakova zarizeni - rozdil, cachovani, priklady
60. Jak funguje FAT?
61. Jak funguje EXT2?
62. Jak funguje NTFS?
63. Rozdil mezi VFS a IFS
64. Co je to IRP?
65. Princip Fast IO
66. Obecny princip IO sybsitemu - sekvenci cinnosti toho co se deje kdyz chce proces napr zapsat na disk
67. Kolik ruznych zarizeni muzeme pouzit pri (teoreticky) kaskadovem zapojeni 3 PICu?
68. K cemu slouzi Message-Signaled Interrupts?
69. Princip APICu
70. Co to jsou virtualni IRQ
71. Jaky je rozdil mezi APIC a IO APIC?
72. Rozdil mezi top half a bottom half
73. Rozdil mezi SoftIRQ a Teskled
74. K cemu je Workqueue
75. Polling vs IRQ
76. Princip IO_uring
77. Princip DMA, jak je DMA pouziva adresovani? Jak se to resi adresovani v pripade legacy modu a long modu?
78. Co je to emulace?
79. Co je to virtualizace?
80. Typy VMM
81. Princip V86
82. Jak velky kus pameti musime alokovat kdyz chceme virtualizvat real-mode?
83. Kdy je vyhazovani vyjimky povazovano za zcela korektni a chtene chovani?
84. Co je to paravirtualizace?
85. Princip Binary Translation
86. Co to je VT-x a jeho princip
87. K cemu se pouziva VPID
88. K cemu slouzi shadow page table
89. Princip VfA a jeho realizace
90. Priklady kdyz potrebujeme virtualizaci (3)