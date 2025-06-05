# I found out the hard way #

## Version conflict when using PG_DUMPALL
Hey all

In order to make my Windows Server to open with codepage 1252, I used below code
``` PowerShell
Get-ItemProperty -Path 'HKLM:\Software\Microsoft\Command Processor' -Name "Autorun" 
Set-ItemProperty -Path 'HKLM:\Software\Microsoft\Command Processor' -Name "Autorun" -Value "CHCP 1252"
Remove-ItemProperty -Path 'HKLM:\Software\Microsoft\Command Processor' -Name "Autorun" 
```
Today (June 5th 2025) I finally discovered that the use of AUTORUN made 'pg_dumpall' to show this:
``` CMD
pg_dumpall: error: program "pg_dump" was found by "C:/Program Files/PostgreSQL/17/bin/pg_dumpall"
but was not the same version as pg_dumpall
```
This error message is way out wrong. As soon as you remove the AUTORUN key in HKLM, the pg_dumpall will work again.

It has nothing to do with the value of AUTORUN. It is the existence of the key, that triggers the error.
