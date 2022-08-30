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
%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```
