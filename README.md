# h6 Tehtävä

## Kone

- MacBook Air(2015)
- Intel i5 1,6 GHz Dual-Core prossu
- 8 GB RAM
- macOS Monterey v.12.6.2

## x)

### Apache Software Foundation 2023: Getting Started

- "Client" eli käyttäjän verkkoselain muodostaa yhteyden palvelimeen ja pyytää resursseja URL-polun avulla.
- Palvelin lähettää vastauksen, joka koostuu palvelimen tilakoodista(Status code) sekä vastauksen rungosta(Response body). Status code kertoo onnistuiko pyyntö vai ei, mikäli ei niin se kertoo millainen virhe ilmaantui.
- Palvelimeen yhdistääkseen "client:n" on ensin määriteltävä palvelimen nimi IP-osoitteeksi muodostaakseen yhteyden palvelimeen.  
- Apache HTTP palvelin konfiguroidaan käyttämällä yksinkertaisia tekstitiedostoja, joiden määrityskäskyt voivat olla yleisiä tai tiettyä hakemistoa koskevia. Jos haluaa, että määrityskäskyt ovat yleisiä laitetaan ne ``<Directory>, <Location>, <VirtualHost>`` ulkopuolelle ja tiettyyn hakemistoon kohdistaessa ``<Directory>`` sisään.
- DocumentRoot määrittää mihin tiedostojärjestelmään staattinen sisältö sijoitetaan.(HTML files, image files, CSS files)
- Apache pavelimen järjestelmänvalvojana arvokkaita ovat lokitiedostot sekä erityisesti virheloki.

### Apache Software Foundation 2023: Name-based Virtual Host Support

- IP-pohjainen isännöinti(IP-based hosting), vaatii erilliset IP-osoitteet jokaiselle isännälle(host).
- Nimiin perustuva isännöinti(Name-based hosting) jakaa saman IP-osoitteen useiden isäntien(host) kesken ja on täten usein simppelimpi ratkaisu.
- Name-based virtual host valitsee vain sopivimman name-based virtual hostin, kun ehdokkaat on rajattu parhaaseen IP-pohjaiseen vastaavuuteen.
- Pyynnöstä palvelin löytää tarkimman vastaavan ``<VirtualHost>`` argumentin pyynnön käyttämän IP-osoitteen ja portin(port) perusteella. Jos useampi virtuaalinen host sisältää vastaavan IP-osoitteen ja portin, Apache vertaa seuraavaksi ``<ServerName>`` ja ``<ServerAlias>`` komentoja pyynnössä olevaan palvelimen nimeen. Jos vastaavuutta ei löydy valitsee palvelin ensimmäisen vaihtoehdon, joka vastaa IP-osoitteen ja portin perusteella.

## a) 

- Käynnistin apache2 palvelimen komennolla ``$ sudo systemctl start apache2``. Etusivu toimi normaalisti.

- Aloitin uuden etusivun luomisen muokkaamalla uuden tiedoston ``/etc/apache2/sites-available/`` kansioon. Nimesin tiedoston ``frontpage.conf``.
Komento:

        $ sudoedit /etc/apache2/sites-available/frontpage.conf

- ``sudoedit``komennon avulla pääsin muokkamaan suoraan tiedostoa ``micro`` editorilla, jotta palvelin toimii uuden luomani sivun kautta. Kirjoitin sivun tiedostoon tiedostopolun, josta palvelin hakee sivun.

![Add file: frontpage](cat-frontpage.png)

- Otin käyttöön sivuista uuden luomani frontpage.conf sivun. Ensin katsoin saatavilla olevat sivut komennolla ``$ ls /etc/apache2/sites-available/``

![Add file: Sivut](sites-available.png)

- Otin käyttöön uuden sivun komennolla ``$ sudo a2ensite frontpage.conf``.
- Tämän jälkeen otin vanhan etusivun pois käytöstä komennolla ``$ sudo a2dissite 000-default.conf``, jonka jälkeen käynnistin apache2 palvelimen uudelleen komennolla ``$ sudo systemctl restart apache2.service``.

- Siirryin kotihakemistoon tekemään vielä kansion ja html tiedoston sivua varten. Siirryin kotihakemistoon komennolla ``cd`` ja tarkistin polun vielä ``pwd`` komennolla. Tein kansion public_sites komennolla ``$ mkdir public_sites`` ja siirryin kansioon ``$ cd public_sites/``. 
- Tein kansioon komennolla ``$ micro index.html`` tiedoston, johon kirjoitin simppelin lauseen, joka tulee uudelle etusivulle näkyviin.

![Add file: Uusi Etusivu](index-micro.png)

Tallensin tiedoston ja menin takaisin kotihakemistoon(``$ cd``) ja testasin uutta etusivua komennolla ``$ curl 'http://localhost``. Palvelin vastasi juuri kirjoittamallani tekstillä, että uusi etusivu toimii.

![Add file: Etusivu test](curl-etusivu.png)

Testasin vielä verkkoselaimessa:

![Add file: Etusivu test2](web-etusivu.png)

## b)

Apache2 palvelimen etusivu vastasi seuraavasti.

![Add file: Error](error.png)

Lähdin selvittämään vikaa katsomalla ensin apache2:n ``error.log`` komennolla ``$ sudo tail -1 /var/log/apache2/error.log``. Sain error lokista seuraavan vastauksen.

![Add file: Error log](error-log.png)

Lokissa näkyi aikaleimalla varusteltu ``error`` tason viesti `` AH01630: client denied by server configuration: /home/miikkas/public-sites``, joka viittaisi vahvasti siihen, että palvelimen konfiguraatio ei onnistunut /home/miikkas/public-sites kautta. 
Kokeilin laittaa komentokehotteeseen kyseisen tiedostopolun:

![Add file: tiedostopolku](sivupolku.png)

Niinkuin vastauksesta näkee, komentokehote ei löydä kyseistä tiedostopolkua ``No such file or directory``. Etsin sitten kotihakemistosta kansiota syöttämällä ``cd public`` ja painamalla ``Tab`` näppäintä kahteen kertaan, jolloin komentokehote antoi vaihtoehdoiksi kansiot ``public_html`` ja ``public_sites``. Eli tiedostopolku on kirjoitettu väärin, public_sites kansio oli kirjoitettu väliviivalla alaviivan sijaan. Sen pitäisi olla ``.../public_sites`` eikä ``.../public-sites``

Korjasin virheen, jonka jälkeen etusivu toimi jälleen halutulla tavalla.

Kävin katsomassa myös ``apache2ctl configtest`` tuloksia, josta löytyi:

![Add file: Configtest](configtest.png)

Mielestäni tätä kautta sai paljon selvemmän kuvan ongelmasta. ``configtest`` tarjosi suoraan, että ``DocumentRoot [/home/miikkas/public-sites/] does not exist``, joka kertoo suoraan, että kyseinen teidostopolku ei ole olemassa. Error osoittaa, että vika on selkeästi tiedostopolussa, sillä lopussa lukee ``Syntax OK``, joka kertoo ettei tiedostossa ole kirjoitusvirheitä. 

Molemmat virheilmoitukset antavat virhekoodit, joiden avulla voisi lähteä vikaa myös etsimään. ``configtest`` ei kuitenkaan tarjoa esimerkiksi aikaleimaa, joka auttaa miettimään milloin ja miten virhe olisi voinut tapahtua.

## Lähteet

- The Apache Software Foundation 2023, Getting Started, Luettavissa: https://httpd.apache.org/docs/2.4/getting-started.html
- The Apache Software Foundation 2023, Name-based Virtual Host Support, Luettavissa: https://httpd.apache.org/docs/current/vhosts/name-based.html
- Karvinen Tero, https://terokarvinen.com/2023/linux-palvelimet-2023-alkukevat/#h6-based
