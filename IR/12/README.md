- extrahovani semantickych informaci z multimedialnich dat
    - audio, video, obrazky, ...
- metody
    - extrahovani priznaku
        - sumarizace obsahu media
        - detekce vzoru (patterns)
    - filtrovani
    - kategorizace

- MMIR (multimedia information retrieval) ukoly
    - analyza bioinfmaci (DNA sekvence)
    - zpracovani biosignalu (EKG)
    - content-based image nebo video retrieval
    - rozpoznani obliceje
    - audio klasifikace
    - rozpoznani reci
    - prohlizeni videa (video browsing)

- Content-based image retrieval (CBIR)
    - take zname jako
    - "query by image content" (QBIC)
    - "content-based visual information retrieval" - ((CBVIR))
    - vs. concept-based pristupy
        - idexovani klicovych slov, popisku
        - high-quality anotace je podstatna
            - chybejici ibrazky - synonyma?
            - bezpecnostni kamery
    - obsah: barvy, tvary, textury

- CBIR techniky (1)
    - dotaz prikladem
        - vybrany jiz existujici obrazek
        - uzivatel nakresli pribliznou aproximaci obrazku
    - vyzaduje lidskou zpetnou vazbu
        - zpresneni vysdku pomoci oznaceni jak relevantni a nerelevantni
    - machine learning

- CBIR techniky (2)
    - porovnani obsahu s pouzitim mereni vzdalenosit obrazku
        - vzdalenost obrazku meri podobnost obrazku na zaklade nekolika dimenzi
            - barva (histogram)
            - textury (vzory)
            - tvary (musime pouzit detekci hran, segmentaci, ...)
    - vyhodnoceni ziskane informace
        - zakladni evaluace (precision & recall)

- Aplikace CBIR
    - architektonicky a engineersky design
    - prevence zlocinu (bezpecnostni kamery)
    - geograficke informace
    - vojensky prumysl
    - Nudity-detection
    - rozpoznani obliceje
        - system schopny identifikovat a verifikovat osobu z digitalniho obrazku nebo video framu 
        - zameruje se na specificke risy
            - pozice oci, nosu, ust, textura kuze, celisti, ...
        - pouzito v bezpecnostnich systemech a marketingu

- Music Information Retrieval (MIR)
    - kategorizovat, modifikovat a dokonce i vytvorit muziku
    - oddeleni tracku a muzikalnich instrumentu - karaoke
    - pouzito v bezpecnostnich systemech a marketingu - MIDI soubor z audia
    - rozpoznani muziku (shazam)

- Rozpoznani reci
    - automatic speech recognition / computer speech recognition / prevedeni reci na text
    - identifikace speakera
        - pouziti spravnych trenovacich dat
        - autentizace
    - Supervised ML (uceni s ucitelem)