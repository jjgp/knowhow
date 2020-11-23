# OpenSSH for Windows

## Adding OpenSSH server

I had the OpenSSH client but not the server as evident by the output of
`Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'`. To add the server:

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```
## Starting the OpenSSH daemon

```powershell
Start-Service sshd # start service
Get-Service sshd # check status
Set-Service -Name sshd `
    -StartupType 'Automatic' # set the daemon to start automatically
```

## Setting the default shell for OpenSSH

```powershell
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" `
    -Name DefaultShell `
    -Value "$env:WINDIR\System32\WindowsPowerShell\v1.0\powershell.exe" `
    -PropertyType String `
    -Force
```

Alternatively, WSL may be set as the default shell by setting the value to
`C:\WINDOWS\System32\bash.exe`. This will start a bash session in the default
WSL distributions as noted in [5](#references).

## Using public key authentication

I set up [vim](https://www.vim.org/download.php#pc) with a **full**
installation so that I may use it to edit files within powershell. Most of this
setup is referenced from [2](#references).

Uncomment, set, or comment out the following lines by editting the
`sshd_config` with `vim $env:PROGRAMDATA\ssh\sshd_config`:

```powershell
PubkeyAuthentication yes

# later in the file
PasswordAuthentication no
PermitEmptyPasswords no

# comment out the following later in the file
#Match Group administrators
#       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

Note the last part must be commented out to set known public keys within the
`$HOME\.ssh\authorized_keys` file. To create this file:

```powershell
mkdir "$HOME\.ssh"
New-Item “$HOME\.ssh\authorized_keys”
vim “$HOME\.ssh\authorized_keys” # to add the known public key
```

Generating a public key may be done with `ssh-keygen -t rsa`. The file
permissions of the `authorized_keys` must be updated. I followed
[2](#references) for this.

```powershell
icacls.exe “$HOME\.ssh\authorized_keys” /remove “NT AUTHORITY\Authenticated Users”
icacls.exe “$HOME\.ssh\authorized_keys” /inheritance:r
Get-Acl “$env:ProgramData\ssh\ssh_host_dsa_key” | Set-Acl “$HOME\.ssh\authorized_keys”
```

# References

1. [How to SSH into a Windows 10 Machine from Linux OR Windows OR anywhere](https://www.hanselman.com/blog/how-to-ssh-into-a-windows-10-machine-from-linux-or-windows-or-anywhere)
2. [Configure SSH Server With Windows 10 Native Way](https://medium.com/rkttu/set-up-your-ssh-server-in-windows-10-native-way-1aab9021c3a6)
3. [OpenSSH Server Configuration for Windows 10 1809 and Server 2019](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_server_configuration?WT.mc_id=-blog-scottha)
4. [Installation of OpenSSH For Windows Server 2019 and Windows 10](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse)
5. [THE EASY WAY how to SSH into Bash and WSL2 on Windows 10 from an external machine](https://www.hanselman.com/blog/the-easy-way-how-to-ssh-into-bash-and-wsl2-on-windows-10-from-an-external-machine)