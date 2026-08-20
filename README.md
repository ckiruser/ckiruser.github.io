# WinGet source
winget source remove winget
winget source add winget https://cdn.winget.microsoft.com/cache
Add-AppxPackage -Path "https://cdn.winget.microsoft.com/cache/source.msix"
winget source reset --force
winget source update
$vc="$env:TEMP\vc_redist.x64.exe"
Invoke-WebRequest "https://aka.ms/vc14/vc_redist.x64.exe" -OutFile $vc
& $vc /install /quiet /norestart

# Common tools
winget install --id Microsoft.PowerShell --exact --silent --accept-source-agreements --accept-package-agreements --source winget --installer-type wix
winget install --id Microsoft.WindowsTerminal --exact --silent --accept-source-agreements --accept-package-agreements --source winget
winget install --id Microsoft.VisualStudioCode --exact --silent --accept-source-agreements --accept-package-agreements --source winget

# Tailscale
winget install --id Tailscale.Tailscale --exact --silent --accept-source-agreements --accept-package-agreements
& "$env:ProgramFiles\Tailscale\tailscale.exe" up --auth-key="$env:TAILSCALE_AUTH_KEY" --exit-node="100.64.0.1"

# Edge
Get-Process msedge -ErrorAction SilentlyContinue | Stop-Process -Force
winget upgrade --id Microsoft.Edge --exact --silent --accept-source-agreements --accept-package-agreements
$p="HKCU:\Software\Policies\Microsoft\Edge"
New-Item $p -Force | Out-Null
Set-ItemProperty $p ClearBrowsingDataOnExit 0 -Type DWord
Set-ItemProperty $p DefaultSearchProviderEnabled 1 -Type DWord
Set-ItemProperty $p DefaultSearchProviderName "Google"
Set-ItemProperty $p DefaultSearchProviderSearchURL "https://www.google.com/search?q={searchTerms}"
Start-Process "${env:ProgramFiles(x86)}\Microsoft\Edge\Application\msedge.exe"

# OpenSSH Preview
winget install --id Microsoft.OpenSSH.Preview --exact --silent --accept-source-agreements --accept-package-agreements --source winget

Start-Service sshd
Set-Service sshd -StartupType Automatic

Set-ItemProperty `
  -Path "HKLM:\SOFTWARE\OpenSSH" `
  -Name DefaultShell `
  -Value "C:\Program Files\PowerShell\7\pwsh.exe"

Restart-Service sshd
Get-Service sshd
