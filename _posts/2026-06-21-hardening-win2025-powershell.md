---
layout: post
title: "Hardening Windows Server 2025 with PowerShell"
date: 2026-06-21
categories: windows security powershell
---
One of the first things I do after deploying a new Windows Server is lock it down before it ever hosts a workload.

Windows Server 2025 ships with solid security defaults, but "secure by default" does not mean "finished." Every environment is different and there are always settings that should be reviewed based on your organization's requirements.

The good news is that nearly every security configuration can be automated with PowerShell. That means you can build a repeatable hardening process instead of clicking through dozens of management consoles every time you deploy a server.

Here's a practical starting point.

## Start with PowerShell 7

Although Windows PowerShell is still included with the operating system, I recommend installing PowerShell 7 for day-to-day administration. It provides better performance, improved scripting features and ongoing development.

You can verify your version with:

```powershell
$PSVersionTable.PSVersion
```

## Keep Windows Updated

Before making configuration changes, install the latest cumulative updates.

```powershell
Install-Module PSWindowsUpdate -Force

Import-Module PSWindowsUpdate

Get-WindowsUpdate

Install-WindowsUpdate -AcceptAll -AutoReboot
```

There's little value in hardening a server that's missing security patches.

## Remove Unnecessary Server Roles

Every installed role increases the attack surface.

List installed roles:

```powershell
Get-WindowsFeature | Where-Object Installed
```

Remove anything the server doesn't need.

For example:

```powershell
Uninstall-WindowsFeature Web-FTP-Server
```

A dedicated file server shouldn't also be running FTP, IIS or other unnecessary services.

## Disable SMBv1

There are very few legitimate reasons to keep SMBv1 enabled.

```powershell
Disable-WindowsOptionalFeature `
    -Online `
    -FeatureName SMB1Protocol `
    -NoRestart
```

Confirm the configuration:

```powershell
Get-SmbServerConfiguration | Select EnableSMB1Protocol
```

## Enable Windows Defender

Microsoft Defender is significantly more capable than it was several years ago.

Verify real-time protection:

```powershell
Get-MpComputerStatus
```

Enable cloud-delivered protection:

```powershell
Set-MpPreference `
    -MAPSReporting Advanced `
    -SubmitSamplesConsent SendSafeSamples
```

Keeping Defender configured properly provides an additional layer against ransomware and malware.

## Configure Windows Firewall

The Windows Firewall should remain enabled on every network profile.

```powershell
Set-NetFirewallProfile `
    -Profile Domain,Private,Public `
    -Enabled True
```

Review existing rules:

```powershell
Get-NetFirewallRule |
Where-Object Enabled -eq True
```

Disable rules you don't actually need.

## Turn On PowerShell Transcription

One of the easiest security wins is logging administrative activity.

Create a directory for transcripts:

```powershell
New-Item `
    -ItemType Directory `
    -Path C:\PowerShellTranscripts `
    -Force
```

Enable transcription through Group Policy or configuration management so every PowerShell session is recorded.

Those logs become incredibly valuable during troubleshooting or security investigations.

## Audit Local Administrators

It's surprisingly common to find unnecessary accounts with administrative privileges.

List local administrators:

```powershell
Get-LocalGroupMember Administrators
```

Review every account carefully.

The fewer privileged accounts you have, the smaller your attack surface becomes.

## Disable Unused Services

Many servers continue running services that aren't required.

Review automatic services:

```powershell
Get-Service |
Where-Object StartType -eq Automatic
```

If a service isn't needed, disable it.

```powershell
Set-Service `
    -Name Fax `
    -StartupType Disabled
```

Be careful here. Always understand what a service does before disabling it.

## Enable BitLocker

If your hardware supports TPM, encrypt the operating system volume.

```powershell
Enable-BitLocker `
    -MountPoint C: `
    -EncryptionMethod XtsAes256 `
    -UsedSpaceOnly
```

Encryption protects data if the server is stolen or a storage device leaves your control.

## Enable Secure Remote Management

PowerShell Remoting should use encrypted connections.

Verify WinRM:

```powershell
Test-WsMan
```

Enable PowerShell Remoting if necessary:

```powershell
Enable-PSRemoting -Force
```

In larger environments, consider using HTTPS listeners instead of HTTP.

## Review Event Logs

Security monitoring is just as important as prevention.

Useful logs include:

```powershell
Get-WinEvent `
    -LogName Security `
    -MaxEvents 50
```

Automating log collection into a centralized SIEM provides much better visibility than checking individual servers.

## Automate Everything

The real advantage of PowerShell isn't that it replaces graphical tools.

It's that your hardening process becomes repeatable.

Instead of documenting dozens of manual steps, you can store your security baseline in source control, review changes through pull requests and apply the same configuration to every new server.

That consistency is often more valuable than any single security setting.

## Final Thoughts

Hardening Windows Server 2025 isn't about applying every recommendation you can find on the internet. It's about reducing unnecessary risk while keeping the server manageable.

PowerShell makes that process repeatable, auditable and easy to improve over time.

Start with a small baseline, automate it, test it thoroughly and expand it as your environment grows. A predictable deployment process is one of the strongest security controls you can have.
