# TEMP — enable SSH on the Windows lab PC (delete after use)

## ▶ RUN ON THE WINDOWS LAB PC — PowerShell **as Administrator**
Copy-paste this whole block once. It installs + starts the OpenSSH **server**, opens the
firewall, and prints the username + IP you send back to me.

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
whoami
ipconfig | findstr /i "IPv4"
```

Then send me:
- the `whoami` line   (e.g. `labpc\student`)
- the IPv4 address    (e.g. `192.168.1.42`)

---

## ▶ I run these on the Linux PC (after you give me user + IP)

```bash
ping -c 2 <WINDOWS_IP>
ssh-copy-id -i ~/.ssh/id_hadoop.pub <user>@<WINDOWS_IP>     # you type the Windows password ONCE
ssh -i ~/.ssh/id_hadoop <user>@<WINDOWS_IP> "whoami"        # confirm passwordless login
```

Once that works I'll drive the Windows box command-by-command to set up **Lab 3**
(matrix multiplication) — Hadoop is already installed there, so we go straight to compile + run.
```
