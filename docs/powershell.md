# PowerShell Cheat Sheet

## Execution policy

Inspect:

```powershell
Get-ExecutionPolicy
Get-ExecutionPolicy -List
```

Common developer-machine policy for the current user:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

Temporary policy for **this PowerShell process only**:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Unblock a downloaded script/file:

```powershell
Unblock-File .\script.ps1
```

Unblock a directory of scripts:

```powershell
Get-ChildItem . -Recurse -Filter *.ps1 | Unblock-File
```

Prefer `RemoteSigned` or one-process `Bypass` over weakening machine-wide policy.

## Run scripts

```powershell
.\script.ps1
& ".\script with spaces.ps1"
```

Arguments:

```powershell
.\script.ps1 -Environment Dev -Verbose
```

## Help / discovery

```powershell
Get-Help Get-Process
Get-Help Get-Process -Examples
Get-Command *service*
Get-Member
```

## Files

```powershell
Get-ChildItem
Get-ChildItem -Recurse
Get-ChildItem -Force
Get-Content .\file.log
Get-Content .\file.log -Tail 100
Get-Content .\file.log -Tail 100 -Wait
```

Search:

```powershell
Select-String -Path .\*.log -Pattern "ERROR"
Get-ChildItem -Recurse -File | Select-String -Pattern "connection refused"
```

Copy/move/remove:

```powershell
Copy-Item source destination
Move-Item source destination
Remove-Item file.txt
Remove-Item directory -Recurse -Force    # DANGER
```

## Environment variables

```powershell
Get-ChildItem Env:
$env:PATH
$env:MY_VAR = "value"
Remove-Item Env:MY_VAR
```

Persist for current user:

```powershell
[Environment]::SetEnvironmentVariable("MY_VAR", "value", "User")
```

## Processes

```powershell
Get-Process
Get-Process -Name dotnet
Stop-Process -Id <pid>
Stop-Process -Name <name>
Stop-Process -Id <pid> -Force
```

## Services

```powershell
Get-Service
Get-Service -Name <service>
Start-Service <service>
Stop-Service <service>
Restart-Service <service>
```

## Network diagnostics

```powershell
Test-NetConnection google.com -Port 443
Test-NetConnection <host> -Port 5432
Resolve-DnsName <hostname>
Get-NetTCPConnection
Get-NetTCPConnection -State Listen
Get-NetTCPConnection -LocalPort 8080
```

Find owning process:

```powershell
Get-NetTCPConnection -LocalPort 8080 |
  Select-Object LocalAddress,LocalPort,State,OwningProcess

Get-Process -Id <OwningProcess>
```

## HTTP

```powershell
Invoke-WebRequest https://example.com
Invoke-RestMethod https://api.example.com/health
```

JSON POST:

```powershell
$body = @{
    name = "test"
} | ConvertTo-Json

Invoke-RestMethod `
  -Method Post `
  -Uri "https://api.example.com/items" `
  -ContentType "application/json" `
  -Body $body
```

## JSON

```powershell
$data = Get-Content .\config.json -Raw | ConvertFrom-Json
$data.someProperty

$obj | ConvertTo-Json -Depth 10 | Set-Content .\out.json
```

## Objects / filtering

```powershell
Get-Process |
  Where-Object CPU -gt 100 |
  Sort-Object CPU -Descending |
  Select-Object -First 10 Name,Id,CPU
```

## Jobs

```powershell
Start-Job { Get-Process }
Get-Job
Receive-Job <id>
Remove-Job <id>
```

## History

```powershell
Get-History
Invoke-History <id>
```

PSReadLine persistent history location:

```powershell
(Get-PSReadLineOption).HistorySavePath
```

Do not put secrets directly in commands if history is enabled.

## Encoding / hashes

```powershell
Get-FileHash .\file.zip -Algorithm SHA256
[Convert]::ToBase64String([IO.File]::ReadAllBytes(".\file.bin"))
```

## Admin check

```powershell
([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).
  IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```
