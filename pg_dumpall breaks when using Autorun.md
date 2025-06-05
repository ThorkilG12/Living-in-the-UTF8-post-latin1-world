# I found out the hard way #
Hej alle
´´´ Powershell
Get-ItemProperty -Path 'HKLM:\Software\Microsoft\Command Processor' -Name "Autorun" 
Set-ItemProperty -Path 'HKLM:\Software\Microsoft\Command Processor' -Name "Autorun" -Value "CHCP 1252"
Remove-ItemProperty -Path 'HKLM:\Software\Microsoft\Command Processor' -Name "Autorun" 
´´´
SÅdan
