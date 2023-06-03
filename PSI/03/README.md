- overview 

    <img src="../images/03/01.png">

    - APDU = application
    protocol data unit
    - PDU – protocol
    data unit

- Transmission Control Protocol - TCP
    - spojove orientovany protokol
    - client-server model
        - ip adresa + port
    - full duplex, spolehlivy prenost dat
        - data jsou prenasene po segmentech (datova jednotka)
        - nastaveni spojeni
        - prenos, acknowledgments, retransmissions kdyz se ztrati data
        - ukonceni spojeni 

         <img src="../images/03/02.png">

         - code bits

            <img src="../images/03/03.png">

        - TCP header checksum - stejne jako u UDP je spocitan pres pseudoheader pridaneho segmentu

            <img src="../images/03/04.png">

    - aplikacni rozhrani

        <img src="../images/03/05.png">

    - TCP API (BSD sokety)

        <img src="../images/03/06.png">

    - TCP navazani spojeni  (3-way handshake)

        <img src="../images/03/07.png">

        <img src="../images/03/08.png">

    - rizeni toku dat

        <img src="../images/03/09.png">

        - nastaveni poctu bytu tak aby prijemce nebyl prilis vytizen 
        - ACK number = cislo dalsiho bytu ktery oceva ze prijme

        <img src="../images/03/10.png">

        <img src="../images/03/11.png">

        <img src="../images/03/12.png">

        <img src="../images/03/13.png">

        <img src="../images/03/14.png">

        - data nejsou prenesena do vyssi vrstvy hend; buffer je vyprazdnen az po dosazeni urcite velikosti
        - realitime aplikace 
            - data nemuzou cekat v bufferu
            - hned se preposlou (remote terminal)
            - PSH (push flag) rika prijemci ze data musi byt prenesena do vyssi vrstvy okamzite

        - TCP - ukonceni komunikace
            - 4-way

                <img src="../images/03/15.png">

            - 3-way

                 <img src="../images/03/16.png">

            - soucasne

                 <img src="../images/03/17.png">

        - TCP connection je defakto stavovy automat
        - protokoly pres TCP
            - HTTP (80), SSH (22), SMTP (25), Telnet (23), FTP (20 - data; 21 - control)

- Internet Protocol Version 6
    - proc IPv6?
        - rozsireni adresni prostor (mel by byt dostatecne velky naporad)
        - 3 typy adres
            - Unicast - pro jedno rozhrani
            - Multicast - pro mnozinu rozhrani na stejnem fyzickem mediu (paket je poslan na vsechny interface svazane s danou IPv6 adresou)
            - Anycast - pro mnozinu rozhrani na jinem fyzikalnim mediu (paket je poslat pouze na jeden interface, ne na vsechny)
        - unifikovane schema adresovani pro internetove a privatni site
        - hierarchicke routovani 
        - zvysena bezpecnost
            - zabudovano do IPv6 (sifrovani, autentizace, data path tracing)
        - quality of service (QoS) support
        - optimilazece pro rychle switchovani / routovani
        - podpora mobilnich zarizeni
        - hladky prechoz z verze 4 na verzi 6

    - adresy
        - 128 bitu
        - reprezentace: `fedc:ba98:7654:3210:fedc:ba98:7654:3210`
        - zkracovani
            - `0123:0000:0000:0000:4567:0000:0000:0000`
            - `123::4567:0:0:0`
            - `123:0:0:0:4567::`
            - `123::4567::` - NEJEDNOZNACNE
        - reprezentace v URL
            - rozdil oproti IPv4 protoze `:` urcuje port
            - `http://[fedc:ba98::ba98:7654:3210]:8080/api`

        <img src="../images/03/18.png">

        - `::1/128` - loopback; localhost 
        - `fe80::/10` - local link address
            - fe80[54 zero bits][64 bits interface ID]
            - autokonfigurace
            - neroutuje se
        - zname IPv6 adresy

            <img src="../images/03/19.png">

        - multicast

            <img src="../images/03/20.png">

        - Multicast addresses scopes

            <img src="../images/03/21.png">

            <img src="../images/03/22.png">

    - fragmentace
        - postupne eliminovana
        - nepodporovana IPv6 routry => musi resit endpointy samostatne (path MTU discovery)
        - MTU = minumal translation unit

- ICMPv6
    - narozdil oproti IPv4 neni pouzit jen pro reportovani chyb
    - nahraduje take napriklad protokoly ARP, RARP - Neighbor Discovery
    - IGMP - multicast group management

    <img src="../images/03/23.png">
