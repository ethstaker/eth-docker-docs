---
title:  Windows
sidebar_position: 12
sidebar_label: Windows
---

Windows may seem like an "easy button". For Eth Docker, it is anything but, and even running native Windows clients
presents multiple challenges. They can all be overcome, and the
[eth-wizard project](https://github.com/stake-house/eth-wizard) aims to do just that.

If you wish to run Eth Docker on Windows regardless, this is what's required.

- Windows 11 Pro 24H2 or later, ideally with 64 GiB RAM so that WSL defaults to 30 GiB
- [WSL 2](https://learn.microsoft.com/en-us/windows/wsl/about), the "Windows Subsystem for Linux", which runs a Linux
kernel in a lightweight VM
- WSL networking that is reachable from the LAN
- Functioning time sync

> I previously suggested Docker Desktop, and no longer do

These are the configuration steps:

Windows
- Verify you are running Windows 11 Pro 24H2 or later and have sufficient RAM
- To keep the system secure, configure Windows Update to download and apply patches automatically, and to update WSL.
Settings -> Windows Update -> Advanced, enable "Receive updates for other Microsoft products" and "Get me up to date".

[WSL Settings](https://learn.microsoft.com/en-us/windows/wsl/wsl-config)
- Configure WSL for [mirrored networking](https://github.com/microsoft/WSL/releases/tag/2.0.0). Edit `.wslconfig` in
your Windows home directory and add
```
[wsl2]
networkingMode=mirrored
```
- Mirrored networking shares the MAC address, IPv4 address and IPv6 address of the Windows host machine. On your
router, set a DHCP reservation for this machine so WSL always has the same local IP; or configure Windows with a static
IP. This makes port forwarding of the P2P ports possible, and makes remote access easier.
- Check memory assigned to WSL with `free -h`. If it's too low for your chosen client mix, edit `.wslconfig` in your
Windows home directory and add a memory section and disable swap, for example
```
[wsl2]
memory=32GB
swap=0
```
- Set the default disk size to 2TB. Alternatively you could increase size after installation. In `.wslconfig` again
```
[wsl2]
defaultVhdSize=2199023255552
```

WSL
- From Windows Store, install WSL and Ubuntu current LTS. Debian is also an option, it is however quite bare-bones
without even man-db out of the box.
- This defaults to WSL 2, but if you have an older WSL 1 install, find it with `wsl --list -v` and change it with
`wsl --set-version DISTRO-NAME 2` as well as `wsl --set-default-version 2`.
- If this was installed before setting a defaultVhdSize, increase the disk space available to WSL
[from 1TB to 2TB](https://learn.microsoft.com/en-us/windows/wsl/disk-space).
- Create a scheduled task in Task Scheduler to keep Ubuntu/Debian in WSL updated.
  - Call it WSLUpdate
  - Run every day at a time you like
  - Run only if any network is connected
  - Run as soon as possible if a start was missed
  - Stop task if it runs longer than 1 hour
  - Create two "Start Program" actions
    - The first is `wsl.exe -u root -e apt-get update`
    - The second is `wsl.exe -u root DEBIAN_FRONTEND=noninteractive apt-get -y --autoremove dist-upgrade`
- Enable `systemd` inside the WSL installation. `nano /etc/wsl.conf` and set it to
```
[boot]
systemd=true
```
Close your WSL windows and in Powershell, run `wsl --shutdown`. When it launches again, systemd should be running.
- Install chrony with `sudo apt install -y chrony`.

Time sync
- Fix Windows time sync if your machine is not domain-joined
  - Change w32time to [start automatically](https://docs.microsoft.com/en-us/troubleshoot/windows-client/identity/w32time-not-start-on-workgroup). In Administrator cmd, but **not** PowerShell, `sc triggerinfo w32time start/networkon stop/networkoff`. Verify with `sc qtriggerinfo w32time`. To get into cmd that way, you can start Admin PowerShell and then just type `cmd`.
  - In `Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\Config`, set `MaxPollInterval` to hex `c`, decimal `12`.
  - Check `Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\Parameters\NtpServer`. If it ends in `0x9` you are done. If it ends in `0x1` you need to adjust `SpecialPollInterval` in `Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\w32time\TimeProviders\NtpClient` to read `3600`
  - Reboot, then from Powershell run `w32tm /query /status /verbose` to verify that w32time service did start. If it didn't, check triggers again. If all else fails, set it to Automatic Delayed startup

- If despite chrony, you still see [clock skew](https://github.com/microsoft/WSL/issues/10006) in WSL, set a scheduled
task to keep WSL in sync with your Windows clock. From non-admin Powershell, run
`schtasks /Create /TN WSLTimeSync /TR "wsl -u root hwclock -s" /SC ONEVENT /EC System /MO "*[System[Provider[@Name='Microsoft-Windows-Kernel-Power'] and (EventID=107 or EventID=507) or Provider[@Name='Microsoft-Windows-Kernel-General'] and (EventID=1)]]" /F`.

Docker
- Use the [QuickStart](../Usage/QuickStart.md) instructions to install Eth Docker and get Docker-CE installed in Ubuntu. Log out and back in so your user can run Docker commands.

Start on boot
- Your node needs to run after Windows reboot for 24/7 uptime. WSL only starts well with a logged-in user.
To solve this, use [Windows ARSO](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/component-updates/winlogon-automatic-restart-sign-on--arso-).
  - Start group policy editor, find "Computer Configuration > Administrative Templates > Windows Components > Windows sign in Options"
and enable "Sign-in and lock last interactive user automatically after a restart"
  - If you are not using Bitlocker, you may also need "Configure the mode of automatically signing in and locking last interactive user after a restart or cold boot".
I was unable to configure this from the GUI and ended up using RegEdit. Navigate to
`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System` and create a new DWORD called
`AutomaticRestartSignOnConfig`. Set it to `0` if you use BitLocker, and to `1` if you are not.

QoL
- Optional: Improve your WSL experience with [Windows Terminal and oh-my-zsh](https://gist.github.com/zachrank/fc71ed301e9823264ddac4fb77975735)
- Optional: Use sparse VHD for WSL, `wsl.exe --list` and then `wsl.exe --manage DISTRO-NAME --set-sparse true`. I
have not tested the performance impact of this.
- Optional: Configure your Windows drive to be [encrypted with Bitlocker](https://www.windowscentral.com/how-use-bitlocker-encryption-windows-10).
Be very careful to print out the recovery key and keep it safe. Always suspend Bitlocker before doing a UEFI/BIOS
upgrade.
