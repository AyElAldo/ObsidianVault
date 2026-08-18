
## Instalación de rol

### Opciones de controlador de dominio
Nivel funcional del bosque debe ser el mas bajo de la red para que sea compatible.

Contraseña para cada DC.

### Comandos en PowerShell
```powershell
#
# Script de Windows PowerShell para implementación de AD DS
#

Import-Module ADDSDeployment
Install-ADDSForest `
-CreateDnsDelegation:$false `
-DatabasePath "C:\Windows\NTDS" `
-DomainMode "WinThreshold" `
-DomainName "dom.miempresa.com" `
-DomainNetbiosName "DOM" `
-ForestMode "WinThreshold" `
-InstallDns:$true `
-LogPath "C:\Windows\NTDS" `
-NoRebootOnCompletion:$false `
-SysvolPath "C:\Windows\SYSVOL" `
-Force:$true
```
