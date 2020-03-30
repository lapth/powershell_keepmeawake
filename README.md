# PowerShell - Keep computer awake and start/stop service
The following code is to Keep computer always awake and stop/start "VPN Check Point" service

## Step 1: Create ps files
- Located them anywhere you want
- Please replace ```$endpointUri="https://IP:Port/$($userId)"``` by your real <IP:Port> for status checking. You can use https://www.mockable.io/ for testing. The expected return value of API/Get is in **text/plain with 1 or 0**

### File: `KeepMeAwake.ps1`
```ps
$userId=$args[0]
$delayS=$args[1]
$printLog=$args[2]
if ($userId -eq "" -or $userId -eq $null) {
	Write-Output "Usage: .\KeepMeAwake.ps1 <UserId> [Interval time] [Enable debug log]"
	Write-Output "Interval time: delay time between 2 checking times, default value is 300 seconds"
	Write-Output "Debug log: 1 means print debug log, 0 means do not print debug log"
	Write-Output "Example: .\KeepMeAwake.ps1 LapTH 300 1"
	exit
}
if ($delayS -eq $null) {$delayS = 5*60}
if ($printLog -eq $null) {$printLog = 0}

Write-Output "Running with: UserName [$($userId)] - Interval time [$($delayS)] - Debug log [$($printLog)]"

$endpointUri="https://IP:Port/$($userId)"
Write-Output "The endpoint for VPN service status checking: $($endpointUri)"

$wsh = New-Object -ComObject WScript.Shell
while (1) {
  if ($printLog -eq 1) {Write-Output "Start processing ..."}
  
  $wsh.SendKeys('+{F15}')
  $isStopService=(Invoke-WebRequest -Uri $endpointUri).Content
  if ($isStopService -eq 1) {
	Write-Output "Trying stop Check Point service"
    Get-Service TracSrvWrapper | Where {$_.status -eq 'running'} |  Stop-Service
  }
  
  if ($printLog -eq 1) {Write-Output "Processed, waiting for $($delayS) seconds"}
  Start-Sleep -seconds $delayS
}
```

### File `StartVPNService.ps1`
```ps
Write-Output "Trying start VPN service"
Get-Service TracSrvWrapper | Where {$_.status -eq 'Stopped'} |  Start-Service
```

## Step 2: Enable Execution Policy
- Run PowerShell as Administrator
- Run  this command

```ps
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine
```

## Step 3: Run the PS file on openned PowerShell
```
> .\KeepMeAwake.ps1 LapTH 300 1
```

=> Done

## Step 4: If you want to start the service, just run the ```StartVPNService.ps1``` as administrator
