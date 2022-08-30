## Privilege Escalation

Käyttöoikeuksien nostaminen tapahtuu, kun ensimmäinen jalansija ja alhaisen tason käyttäjätunnus on saatu haltuun.
Tavoite on saada lisää oikeuksia ja päästä mahdollisesti liikkumaan kohdejärjetelmän osiin, joihin ensimmäisellä kaapatulla käyttäjätunnuksella ei välttämättä ole oikeuksia.
Ideaalitavoite on saada root- (Linux) tai Administrator- (Windows) tason oikeudet.

Käyttöoikeuksia voidaan nostaa monella tavalla, riippuen kohdejärjestelmästä.
Usein käyttöoikeuksien nostamisen mahdollistaa väärin konfiguroidut käyttöoikeudet tai haavoittuvat järjestelmän osat.

**Kohdejärjestelmän kartoitus**
```bash
hostname ## Voi löytyä tietoa järjestelmän käyttötarkoituksesta
uname -a ## Kernel -versio
cat /etc/issue ## Voi löytyä tietoa käyttöjärjestelmästä 
cat /proc/version ## Tietoa prosesseista, kernelistä, ja esim. onko compiler (esim. gcc) asennettu
ps ## Process Status, pyörivät prosessit. "ps -A" = kaikki, "ps axjf" = prosessipuu
env ## Ympäristömuuttujat, esim. PATH
sudo -l ## Kaikki komennot, jota käyttäjä voi ajaa sudona
ls -la ## Normaali ls ei näytä piilotettuja tiedostoja
id ## Käyttäjän tiedot ja ryhmä
ifconfig ## Verkkoliikennetietoja
netstat ## Muodostetut yhteydet, "netstat -a" = Kaikki, "netstat -l" = Yhteydettä odottavat. Voi lisätä t (TCP) tai u (UDP), esim. "netstat -at"
```
Find -komentoja (lisää loppuun 2>/dev/null, siistii virheviestit pois)
```bash
find / -name flag1.txt ## Etsi tiedosto flag1.txt
find / -type d -name config ## Etsi kansio config
find / -type f -perm 0777 ## Luku, kirjoitus, suorita -tiedostot kaikille käyttäjille
find / -perm a=x ##  Suoritettavat tiedostot
find / -perm -u=s -type f ## SUID -bittiset tiedostot 
```

**Kernel-haavoittuvuudet**

**Sudo-haavoittuvuudet**

-- Lisää tulossa --
