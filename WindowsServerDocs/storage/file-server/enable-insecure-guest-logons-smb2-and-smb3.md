---
title: Enable insecure guest logons in SMB2 and SMB3 for Windows client and Windows Server
description: This article describes how to enable guest logons policy in SMB2 and SMB3 for Windows client and Windows Server devices using Group Policy and PowerShell.
ms.topic: how-to
ms.author: wscontent
author: xelu86
ms.date: 05/15/2024
---

# How to enable insecure guest logons in SMB2 and SMB3

This article describes Server Message Block (SMB) insecure guest logon default behaviors, why you might enable guest access, and how to enable it for the SMB client using Group Policy and PowerShell.

Since Windows 2000, Windows disabled inbound guest access and prevented SMB2 and SMB3 client guest authentication since Windows 10. However, guest credentials may still be required when connecting to a third-party device that doesn't support a username and password. The recommendation is to upgrade or replace any third-party software or devices that only support guest authentication.

## Default behaviors

Starting from Windows 10, version 1709 and Windows Server 2019, SMB2 and SMB3 clients no longer allow the following actions by default:

- Guest account access to a remote server.
- Fall back to the Guest account after invalid credentials are provided.

SMB2 and SMB3 have the following behavior for different versions of Windows:

- By default, guest credentials can't be used to connect to a remote share in Windows 10 Enterprise, Windows 10 Pro for Workstations, and Windows 10 Education even if requested by the remote server.

- The use of guest credentials to connect to a remote share is no longer allowed by default in Windows Server 2019 Datacenter and Standard editions, even if requested by the remote server.

- Windows 10 Home and Pro editions still allow the use of guest authentication by default, as they did previously.

- In Windows 11 Pro Insider Preview build 25267, and all subsequent builds, guest credentials can't be used to connect to a remote share by default, even if requested by the remote server.

> [!NOTE]
> This behavior is present in various versions of Windows 10, including 1709, 1803, 1903, 1909, 2004, 20H2, and 21H1, as long as [KB5003173](https://support.microsoft.com/topic/may-11-2021-kb5003173-os-builds-19041-985-19042-985-and-19043-985-2824ace2-eabe-4c3c-8a49-06e249f52527) is installed.

## Reason for enabling guest logons

Enabling guest logons may be necessary where a user needs to access a resource on a server but the server doesn't provide user accounts, only guest access.

> [!CAUTION]
> It's important to note that enabling guest logons can pose a security threat as it permits an attacker to deceive a user into connecting to a spoofed malicious server without producing credential errors or prompts, and executes malicious code such as ransomware. As a result, it's recommended to enable guest logons only in specific situations where they are required.

## Prerequisites

Before you can begin modifying insecure guest logons for the SMB client, you need the following.

- An account that's a member of the Administrators group, or equivalent.
- SMB client running on one of the following operating systems:

  - Windows 10 or later.
  
  - Windows Server 2019 or later.

If you're planning on to enable auditing for insecure guest logons, the SMB client must be running on one of the following operating systems.

- Windows 11 Insider Preview Build 25267 or later.
- Windows Server Insider Preview Build 25991 or later.

## Enable insecure guest logons

Enabling insecure guest logons can be performed through Group Policy or PowerShell.

> [!NOTE]
> If you need to modify the Active Directory domain-based group policy, use **Group Policy Management** (gpmc.msc).

# [Group Policy](#tab/group-policy)

1. Select **Start**, type **gpedit.msc**, and select **Edit group policy**.
1. In the left pane under **Local Computer Policy**, navigate to **Computer Configuration\Administrative Templates\Network\Lanman Workstation**.
1. Open **Enable insecure guest logons**, select **Enabled**, then select **OK**.

# [PowerShell](#tab/powershell)

Run the following command through an elevated PowerShell prompt:

```powershell
Set-SmbClientConfiguration -EnableInsecureGuestLogons $true -Force
```

```powershell
Set-SmbServerConfiguration -EnableInsecureGuestLogons $true -Force
```

---

Both SMB signing, and SMB encryption policies must be disabled in Group Policy in order to use guest logons. Doing so can potentially compromise the security of the client and leave users open to credential theft and relay attacks. Additionally, even if the SMB client is set to allow guest logons, SMB signing, and SMB encryption prevents the use of guest logons.

## Audit insecure guest logons

Once the insecure guest logons policy is enabled, these events are captured in the **Event Viewer**. To review these logs, perform the following steps:

1. Right-click on **Start**, select **Event Viewer**.
1. In the left pane, navigate to **Applications and Service Logs\Microsoft\Windows\SMBClient\Security**.

In the middle pane, you can review the following information concerning these events:

```output
Log Name: Microsoft-Windows-SmbClient/Security
Source: Microsoft-Windows-SMBClient
Logged: Date/Time
Event ID: 3023
Task Category: InsecureGuestLogon
Level: Informational
Keywords: Authentication
User: SYSTEM
Computer:
Description: The SMB client was logged on as Guest account.
```

```output
Log Name: Microsoft-Windows-SmbClient/Security
Source: Microsoft-Windows-SMBClient
Logged: Date/Time
Event ID: 31017
Task Category: RejectedInsecureGuestAuth
Level: Error
Keywords: Authentication
User: NETWORK SERVICE
Computer:
Description: Rejected an insecure guest logon.

             The machine attempted to connect to the server using an
             insecure guest logon. The server denied the connection.
             Ensure that the guest account is enabled on the server
             and configured to allow access from the network.
```

```output
Log Name: Microsoft-Windows-SmbClient/Security
Source: Microsoft-Windows-SMBClient
Logged: Date/Time
Event ID: 31018
Task Category: InsecureGuestAuthEnabled
Level: Warning
Keywords: Authentication
User: NETWORK SERVICE
Computer:
Description: An administrator has enabled AllowInsecureGuestAuth.
             Clients using insecure guest logons are more vulnerable
             to attackers-in-the-middle, phishing, and malware.
```

```output
Log Name: Microsoft-Windows-SmbClient/Security
Source: Microsoft-Windows-SMBClient
Logged: Date/Time
Event ID: 31022
Task Category: AllowedInsecureGuestAuth
Level: Warning
Keywords: Authentication
User: SYSTEM
Computer:
Description: Allowed an insecure guest logon.

             Username: nonexistantaccount
             Server name: 
     
             This event indicates that the server attempted
             to log the user on as an unauthenticated guest and was
             allowed by the client.
```
