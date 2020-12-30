---
layout: post
title: "Terveydenhoitoalan yhtiöiden tietoturvasta"
hidden: false
---
Lokakuun lopussa 2020 psykoterapiakeskus Vastaamoon tehtiin tietomurto. Tietomurron tekijä yritti kirstää Vastaamolta bitcoineja vastineeksi siitä, ettei hän vuotaisi asiakkaiden tietoja Tor-verkkoon. Myöhemmin kiristäjä (tai joku muu) alkoi kiristää myös tietomurron uhreilta rahaa, jotta hän ei julkaisi tietojaan. Ylellä on hyvä yhteenveto tapahtumista [Yle seurasi Vastaamon tietomurtoa](https://yle.fi/uutiset/3-11612399)

Oletettavasti Vastaamon murto tapahtui julkiseen Internettiin avoinna olevast phpMyAdmin-sivusta, jonne pääsi vielä oletussalasanoilla sisään. Oletussalasana on salasana, jota käytetään järjestelmän asentamisen jälkeen. Oletussalasanat ovat yleisesti tunnettuja, ja tarkoituksena on, että ne vaihdetaan heti järjestelmän asennuksen jälkeen. Jostain syystä tämä oli ilmeisesti Vastaamolta jäänyt tekemnättä.

<!--more-->

## Tietoturvan taso ja luokitukset

Vastaamon tietomurron jälkeen monet yritykset vakuuttelivat tietoturvansa korkeaa tasoa, ja kertoivat miten heidän palvelunsa on auditoitu ja kuinka mitään tälläistä ei voi tapahtua. 

Vastaamo kuului valvonnan puolesta Valvirassa ns. B-luokitukseen, eli viranomaiset eivät arvioineet sen tietoturvallisuutta ennen käyttöönottoa. Tähän samaan B-luokaan kuuluu hyvin paljon terveyspalveluita tarjoavia yrityksiä; Työterveyttä, kuntoutusta, terapiaa ja verikokeita tekeviä yrityksiä ym. ym. Lista on pitkä. Yleisesti voidaan todeta, että terveyteen liittyvää tietoa on hyvin paljon näissä B-luokitelluissa järjestelmissä.

## Löydöksiä

Monet yritykset vakuuttivat, että heidän järjestelmän on auditoitu, joten innostuin spkekuloimaan miten hyvin tieto on näissä palveluissa suojattu. Tulokset eivät oleet lupaavia, ja viivästytin tämän julkaisemista parilla kuukaudella, jotta kaikki asianomaiset ehtivät korjatta järjestelmänsä. 

### Vanhat ja turvattomat SSL versiot käytössä

Esimmäinen erikoinen havainto on, että monet näistä sivuista tukevat vielä turvattomia ja vanhoja SSL versioita. Tämän voi todeta ottamalla yhteyttä open-ssl:llä esim.
```
openssl s_client -connect xxxx.fi:443 -tls1_1
CONNECTED(00000003)
```

Tai vastaavasti kannattaisi käyttää [SSL Labsin](https://www.ssllabs.com/ssltest) testiä, joka tuottaa raportin sivustan TLS astuksista. Tämä tarkistaa myös tuetut algoritmit, ja antaa raportin sivuston TLS asetuksista. Monet näistä palveluista menivät katogoriaan B, jota ei mielestäni voida millään tavalla perustella, kun on terveystiedot kyseessä. 

Mikäli käyttäjän järjestelmä on ajantasalla, niin yhteys muodostuu aina turvallisella TLS-versiolla. Palvelun tarjoajan vastuulla onkin estää yhteydet vanhentuneilla (turvattomilla) järjestelmillä. Suomessa kannattaa ottaa esimerkkiä esim. op-ryhmästä, joka lopettaa vanhojen laitteiden ja järjestelmien tuen, jotta asiakkaiden rahat ja tiedot pysyisivät turvassa [Iltalehti: Tapion toimivasta tabletista tuli yhtäkkiä romua](https://www.iltalehti.fi/digiuutiset/a/06c25ed4-ba01-4ed3-9bc6-b86bf70cea1f). Op.fi ei tue TLS:n versioita 1.1, eikä 1.0. Monet suomalaiset joutuvat siis jo verkkopankkia käyttääkseen pitämään järjestelmänsä ajantasalla. Terveysalantoimijoilla ei näinollen voi olla liiketoiminnallista perustetta tukea vanhoja ja turvattomia versioita. 

### Tarkat palvelinversiot palautuvat käyttäjälle

Tämä oli ensimmäinen hälyyttävä havainto. Miltei poikkeuksetta kaikki B-luokitellut järjestelmät palauttivat tarkkoja palvelinversioita käyttäjille. Jos joku oikeasti yrittäisi tehdä tietomurtoa, niin ensimmäinen kiinnostava asia on että millainenkohan järjestelmä tässä olisi takana. 

Tarkat versiotiedot auttavat hyökkääjää melkoisesti, sillä silloin voidaan etsiä tunnettuja haavoittuvuuksia vaikka googlella hakemalla esim. "Microsoft-IIS 7.5 vulnerabilities". Nopeasti löydetään dukumentoituja tietoturva-aukkoja, joita hyödyntämällä järjestelmään voitaisiin tunkeutua. Joukossa oli myös paljon avoimen koodit palvelimia (esim. nginx), josta voidaan tarkalla versionumerolla hakea versionhallinnnasta koodit, ja etsiä mahdollisia haavoittuvuuksia.

Tarkkojen versiotietojen poistaminen on ensimmäisiä muutoksia joita järjestelmään tulisi tehdä heti oletussalasanojen vaihtamisen jälkeen. Tämä ei todellakaan näytä hyvältä! Epäilen, että ainuttakaan B-luokan järjestelmää ei olla auditoitu minkään ulkopuolisen tahon toimesta. Tämä ei olisi jäänyt ainoaltakaan ammattilaiselta huomaamatta.

Alla muutamia esimerkkejä palvelinversioista:
```
Server: nginx/1.8.1
Server: Microsoft-IIS/7.5
Server: nging/1.14.0 + Phusion Passenger 6.0.4
```

### Admin sivuja avoinna julkiseen Internettiin

Vastaamo ei ollut suinkaan ainut, jolla oli Admin paneelit auki julkiseen Internettiin. Löysin avoinna lukuisia olevia phpMyadmin sivuja, sekä ihan käyttäjäystävällisesti kustomoituja adminconsoleita. 

Admin konsolien löytäminen on hyvin helppoa. Riittää, että koitat ladata /phpMyAdmin, tai /admin tai jotain vastaavaa sivua. 

Minun on hieman vaikea ymmärtää, miksi näiden sivustojen hallintaan tarkoitettujen paneelien tulee olla auki julkiseen Internettiin päin? Miksei näihin pääsyä rajata verkkotasolla? Entä jos joku admin-oikeudet omistava henkilö on valinnut huonon salasanan. Esim. salasanan, joka on ollut jo osana aikaisempaa salasanojen vuotoa? Tai mitä jos joku tekeisi saman näköisen sivun, mutta pyrkisi hyvin kohdennetulla "phishing" hyökkäyksellä saamaan admin-käyttäjän salasanan? 

Alla esimerkkejä login-näkymistä. 
![phpmyadmin](/public/images/phpmyadmin.png "phpmyadmin")
![puhtiadmin](/public/images/omapuhti_admin.png "puhti adminconsole")