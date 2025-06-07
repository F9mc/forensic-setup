# Ansible Forensic VM Setup

> Ansible playbook to automate the provisioning of a Windows forensic analysis VM with essential tools and configurations.

---

## Overview

This repository uses Ansible to install and configure a wide range of forensic, analysis, and utility tools on a Windows VM. It leverages Chocolatey for package management, clones key repositories, and sets up custom binaries, PowerShell aliases, and environment variables.

---

## Installed Tools and Packages

### System Utilities (via Chocolatey)
- **git**: Version control system
- **vscode**: Visual Studio Code editor
- **googlechrome**: Google Chrome browser
- **firefox**: Mozilla Firefox browser
- **brave**: Brave browser
- **7zip**: File archiver
- **ripgrep**: Fast search tool
- **powertoys**: Microsoft PowerToys utilities
- **python3-virtualenv**: Python 3 virtual environment tool
- **dbeaver**: Database manager
- **sqlitebrowser**: SQLite database browser
- **bat**: `cat` clone with syntax highlighting
- **exiftool**: Metadata extraction tool
- **wireshark**: Network protocol analyzer
- **openvpn**: VPN client
- **jq**: Json parser

### Python
- **python2**: Installed via Chocolatey (`2.7.18`)
- **python3**: Installed via Chocolatey (`3.11.9`)
- **python3-pip**: Python 3 package manager (updated via pip)
- **uv**: Fast Python package installer (via pip)
- **oletools**: Python malware analysis tools (via pip)

### Forensic and Analysis Tools

#### Memory Analysis
- **Volatility 2**: Cloned from [volatilityfoundation/volatility](https://github.com/volatilityfoundation/volatility) (Python 2)
- **Volatility 3**: Cloned from [volatilityfoundation/volatility3](https://github.com/volatilityfoundation/volatility3) (Python 3)
- **MemProcFS**: (to be added, see TODO)

#### Log & Malware Analysis
- **Zimmerman Tools**: Installed via [Get-ZimmermanTools](https://github.com/EricZimmerman/Get-ZimmermanTools)
- **KAPE**: Cloned from [EricZimmerman/KapeFiles](https://github.com/EricZimmerman/KapeFiles)
  - Zimmerman tools and custom binaries are copied into KAPE's modules folder.
- **SigmaHQ**: [SigmaHQ/sigma](https://github.com/SigmaHQ/sigma) (log detection rules)
- **LogonTracer**: [JPCERTCC/LogonTracer](https://github.com/JPCERTCC/LogonTracer)
- **RegRipper**: [keydet89/RegRipper3.0](https://github.com/keydet89/RegRipper3.0)

#### Malware Analysis
- **Yara**: [VirusTotal/yara](https://github.com/VirusTotal/yara)

#### Mobile Artefacts
- **ALEAPP**: [abrignoni/ALEAPP](https://github.com/abrignoni/ALEAPP)

#### Miscellaneous
- **Sysinternals Suite**: Downloaded from Microsoft
- **Nirsoft Tools**: Downloaded and extracted from [Nirsoft](https://www.nirsoft.net/)
  - Tools are defined in [`roles/nirsoft/vars/main.yml`](roles/nirsoft/vars/main.yml) and provisioned automatically (e.g., BrowsingHistoryView, OutlookAttachView).

#### Additional Tools (via binaries/)
- Any custom or third-party tools placed in the `binaries/` subfolders will be copied to the corresponding `C:\Tool\` directories.

---

## Directory Structure Created

- `C:\Tool\Misc`
- `C:\Tool\Misc\Nirsoft`
- `C:\Tool\MemoryAnalysis`
- `C:\Tool\MalwareAnalysis`
- `C:\Tool\RegistryAnalysis`
- `C:\Tool\MobileArtefacts`
- `C:\Tool\LogsAnalysis`
- `C:\Tool\Zimmerman`
- `C:\Tool\Arsenal Recon`
- `C:\Tool\KapeFiles`
- `C:\Tool\SysinternalsSuite`

---

## File and Folder Copying

All files and subfolders from each `binaries/<folder>` (where `<folder>` is one of `Misc`, `MemoryAnalysis`, `MalwareAnalysis`, `LogsAnalysis`, `Nirsoft`, `Zimmerman`, `Arsenal Recon`, etc.) are copied into the corresponding `C:\Tool\<folder>\` directory on the target Windows machine.


---

## PowerShell Aliases

The following PowerShell aliases are configured for convenience (see [`group_vars/vmwindows`](group_vars/vmwindows)):

- `python3` → `C:\Python311\python3.11.exe`
- `python2` → `C:\Python27\python.exe`
- `grep` → `C:\ProgramData\chocolatey\bin\rg.exe`
- `cat` → `C:\ProgramData\chocolatey\bin\bat.exe`

---

## Nirsoft Role Details

- Tools to be installed are defined in [`roles/nirsoft/vars/main.yml`](roles/nirsoft/vars/main.yml) as a list under `nirsoft_tools`.
- Each tool entry should include at least `name` and `zip_url`.
- The role will:
  1. Check if the tool's `.exe` exists in `C:\Tool\Misc\Nirsoft\<toolname>\`.
  2. Download the zip file to `C:\Windows\Temp\` if missing.
  3. Unzip the tool into `C:\Tool\Misc\Nirsoft\<toolname>\`.
  4. Copy the main `.exe` to `C:\Tool\KapeFiles\Modules\bin\` for easy access by KAPE and other tools.

---

## Setup Instructions

### On the Host Machine

1. **Install Python 3**  
   Ensure Python 3 is installed on your host system.

2. **Install Required Python Packages**  
   Run the following command in the project directory:
   ```bash
   pip3 install -r requirements.txt
   ```

3. **Configure Ansible Hosts**  
   Edit your Ansible inventory file to specify the target hosts and provide the correct credentials.

### On the Target Windows Machine

1. **Enable WinRM**  
   WinRM (Windows Remote Management) must be enabled to allow Ansible to connect.  
   You can use the provided [`setup_winrm.ps1`](setup_winrm.ps1) script at your own risk.

---

### Example Ansible Inventory

Create or edit an `inventory` file in your project directory.  
Below is a template for a Windows VM using WinRM:

```yaml
---
all:
  children:
    vmwindows:
      hosts:
        vm1:
          ansible_host: <WINDOWS_VM_IP>
      vars:
        ansible_connection: winrm
        ansible_winrm_transport: ntlm
        ansible_winrm_server_cert_validation: ignore
        ansible_winrm_port: 5985
        ansible_user: <WINDOWS_USERNAME>
        ansible_password: <WINDOWS_PASSWORD>
```

Replace `<WINDOWS_VM_IP>`, `<WINDOWS_USERNAME>`, and `<WINDOWS_PASSWORD>` with your actual values.

---

## Acknowledgements

This project makes use of the following resources and open-source projects:

- [volatilityfoundation/volatility](https://github.com/volatilityfoundation/volatility)
- [volatilityfoundation/volatility3](https://github.com/volatilityfoundation/volatility3)
- [EricZimmerman/Get-ZimmermanTools](https://github.com/EricZimmerman/Get-ZimmermanTools)
- [EricZimmerman/KapeFiles](https://github.com/EricZimmerman/KapeFiles)
- [SigmaHQ/sigma](https://github.com/SigmaHQ/sigma)
- [JPCERTCC/LogonTracer](https://github.com/JPCERTCC/LogonTracer)
- [keydet89/RegRipper3.0](https://github.com/keydet89/RegRipper3.0)
- [VirusTotal/yara](https://github.com/VirusTotal/yara)
- [abrignoni/ALEAPP](https://github.com/abrignoni/ALEAPP)
- [jq](https://jqlang.org/)
- [Nirsoft](https://www.nirsoft.net/)
- [Introduction to Ansible - Become](https://phelipetls.github.io/posts/introduction-to-ansible/#become)
---

## TODO

- [x] Path env
- [x] Install requirements for Volatility 2/3
- [ ] Get config files
  - [ ] VS Code
  - [?] Browsers
  - [ ] Windows Terminal
  - [ ] PowerToys
- [x] KAPE modules
  - [/] mfiles
  - [x] binaries
- [x] Error handling / depends_on
- [x] Set execution policy
- [x] Install .NET 9
- [x] Copy binary files
- [x] Setup env path
- [x] Change/fail management

---