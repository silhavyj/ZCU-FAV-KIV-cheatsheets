- server muze byt zaroven klientem (napr pri komunikaci s dalsi sluzbou)
- systemy potrebuji pro komunikaci stejny protokol
    - HTTP - textovy protokol
    - MySQL - binarni pres TCP/IP
- klient posila pozadavky, server je zpracovava a odpovida na ne
- HTTP
    - bezstavovy (server si nepamatuje kdo kdy poslal jakej pozadavek - nijak se neovlivnuji)
    - textovy protokol
    - verze 1 - nejvice pouzivana
        - nevyhoda: pro kazde stazeni souboru, poslani pozadavku => jedno TCP spojeni (desitek css, js souboru)
    - verze 2 - umoznuje posilat vice pozadavku pres jedno TCP spojeni
    - verze 3 - bezi ne pres TCP ale UDP

- HTTP request
    ```
    <method> <URI> <version>
    <header1>
    ...
    <headerN>
    <body>

    Example:
    GET /index.html HTTP/1.1
    Host: www.kiv.zcu.cz
    ```
    - Host: na jednom serveru muze byt hostovanych vice "hostu" viz Apache - virtualni hosti 

- typy HTTP metod
    - GET, OPTIONS, HEAD, TRACE - nemeni stav
        - OPTIONS vraci jake metody jsou k dispozici
    - POST, PUT, DELETE - meni stav
- priklady hlavicek v HTTP requestu:
    - host (povinna) - jmeno serveru
    - origin - jmeno serveru ze ktereho by request poslan
    - Content-Type - JSON, HTML, TEXT, ...
    - Authorization - jmeno/heslo

- HTTP response
    ```
    <version> <code> <description>
    <header1>
    ...
    <headerN>
    <body>
    Example:
    HTTP/1.1 200 OK
    Content-Type: text/plain
    Content-Length: 14
    ...
    Hello, world!
    ```

    - typy kodu odpovedi:
        - 200 - OK
        - 204 - OK, ale neposilam ti zpet zadny obsah (napr u POST)
        - 301 - redirect
        - 401 - unauthorized (musis se prihlasit)
        - 403 - forbidden (nemas prava)
        - 404 - stranka nenalezena
        - 500 - chyba na strane serveru

- Cookies:
    - zavadeji stavovost do jinak bezstavoveho protokolu
    - klient odesila cookie spolu s requestem -> server vi ze se jedna o daneho klienta a nacte jeho session (napr z distribuovane cache v pripade horizontalniho skalovani)
    - pouziva se napr i k sledovani uzivatelu - viz reklamy
    - cookie obsahuje session ID
    - muze vest k bezpecnostnim rizikum - ukradeni cookie
    - cookie se nastavuji pres HTTP hlavicky
        ```
        Set-Cookie: name=value [; EXPIRES=dateValue]
        [;DOMAIN=domainName][;PATH=pathName][;SECURE][;...]
        ```
        - secure: odesli pouze pokud se jedna o HTTPS
        - expires: doba platnosti cookie
        - domain: nastavene browserem, napr zcu.cz, kdyz pujdu na FB tak se cookie posilat nebude 
        - path: cesta pro kterou se ma cookie odesilat napr zcu.cz/admin
        - => klient posle vsechny cookie pokud sedi domena, cesta, nevyprsela doba platnosti a pripadne se jedna o HTTPS (ne jen HTTP)

- HTML WebStorage API
    - alternativa misto cookie
    - aplikace si mohou ukladat data v prohlizeni
    - nemusi se posilat na sever s kazdym requestem (narozdil od cookie)
    - pristupne z JS
    - data jsou ulozena dokud nezanikne session/nezavre se tab
        ```
        Window.localStorage
        sessionStorage.setItem('key', 'value');
        let data = sessionStorage.getItem('key');
        sessionStorage.removeItem('key');
        sessionStorage.clear();
        ```
- HTTP Proxy
    - requesty skoro vzdy jdou skrze nejakou proxi
    - Forward
        - requesty do externi site
        - nahrazuje IP addr zroje svoji IP
        - dovoluje restrikci kam se uzivatel pripojuje (napr z firemni site)
        - client → proxy → internet
    - Reverse
        - requesty z externi site
        - slouzi napr jako load balancer
        - jeden vstupni bod pro celou sit
        - client → internet → proxy → content server
    - proxy nahrazuje IP odesilatele svoji vlastni IP adresou
        - nechtene napriklad pri debuggovani
        - nebo napri kdyz je proxy v jinem state a my chceme obsah pro CR
    - informace o odesilateli se predavaji v HTTP hlavicce
        ```
        Forwarded: by=<identifier>;for=<identifier>;host=<host>;proto=<http|https>

        X-Forwarded-For   - puvodni IP klienta
        X-Forwarded-Host  - puvodni IP kam se klient chtel pripojit
        X-Forwarded-Proto - protokol
        ```