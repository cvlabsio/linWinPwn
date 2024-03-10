# linWinPwn - Swiss-Army knife for Active Directory Pentesting using Linux

## Description

linWinPwn is a bash script that wraps a number of Active Directory tools for enumeration (LDAP, RPC, ADCS, MSSQL, Kerberos), vulnerability checks (noPac, ZeroLogon, MS17-010, MS14-068), object modifications (password change, add user to group, RBCD, Shadow Credentials) and password dumping (secretsdump, lsassy, nanodump, DonPAPI). The script streamlines the use of a large number of tools: impacket, bloodhound, netexec, enum4linux-ng, ldapdomaindump, lsassy, smbmap, kerbrute, adidnsdump, certipy, silenthound, bloodyAD, DonPAPI and many others. 

## Setup

Git clone the repository and make the script executable

```bash
git clone https://github.com/lefayjey/linWinPwn
cd linWinPwn; chmod +x linWinPwn.sh
```

Install requirements using the `install.sh` script (using standard account)
```bash
chmod +x install.sh
./install.sh
```

## Usage

### Mode
The linWinPwn script has an interactive mode (default), or automation mode (enumeration only).

**Default: Interactive Mode** - Open interactive menu to run checks separately

```bash
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> [-d <AD_domain> -u <AD_user> -p <AD_password> -H <hash[LM:NT]> -K <kerbticket[./krb5cc_ticket]> -A <AES_key> -C <cert[./cert.pfx]> -o <output_dir>]
```

**Automated Mode** - Using the `--auto` parameter, run enumeration tools (no exploitation, modifications or password dumping)

When using the automated mode, different checks are performed if credentials are provided or not.

- Unauthenticated (no credentials provided)
    - Anonymous enumeration using netexec, enum4linux-ng, ldapdomaindump, ldeep
    - RID bruteforce using netexec
    - kerbrute user spray
    - Pre2k authentication check on collected list of computers
    - ASREPRoast using collected list of users (and cracking hashes using john-the-ripper and the rockyou wordlist)
    - Blind Kerberoast
    - CVE-2022-33679 exploit
    - Check for DNS unsecure updates for AS-REQ abuse using krbjack
    - SMB shares anonymous enumeration on identified servers
    - Enumeration for WebDav, dfscoerce, shadowcoerce and Spooler services on identified servers
    - Check for ms17-010, zerologon, petitpotam, nopac, smb-sigining, ntlmv1, runasppl weaknesses
```bash
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> --auto [-o <output_dir>]
```

- Authenticated (using password, NTLM hash, Kerberos ticket, AES key or pfx Certificate)**
    - DNS extraction using adidnsdump
    - BloodHound data collection
    - Enumeration using netexec, enum4linux-ng, ldapdomaindump, bloodyAD, sccmhunter, rdwatool, sccmhunter, GPOwned
    - Generate wordlist for password cracking
    - netexec find accounts with user=pass 
    - Pre2k authentication check on domain computers
    - Extract ADCS information using certipy and certi.py
    - kerbrute find accounts with user=pas
    - ASREPRoasting (and cracking hashes using john-the-ripper and the rockyou wordlist)
    - Kerberoasting (and cracking hashes using john-the-ripper and the rockyou wordlist)
    - Targeted Kerberoasting (and cracking hashes using john-the-ripper and the rockyou wordlist)
    - SMB shares enumeration on all domain servers using smbmap, manspider and cme's spider_plus
    - Enumeration for WebDav, dfscoerce, shadowcoerce and Spooler services on all domain servers (using cme, Coercer and RPC Dump)
    - Check for ms17-010, ms14-068, zerologon, petitpotam, nopac, smb-signing, ntlmv1, runasppl, certifried weaknesses
    - Check mssql privilege escalation paths
    - Check mssql relay possibilities
```bash
proxychains ./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain>  -d <AD_domain> -u <AD_user> [-p <AD_password> -H <hash[LM:NT]> -K <kerbticket[./krb5cc_ticket]> -A <AES_key> -C <cert[./cert.pfx]>] [-o <output_dir>] --auto
```

### Parameters

**Auto config** - Run NTP sync with target DC and add entry to /etc/hosts before running the modules

```bash
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> --auto-config
```

**LDAPS** - Use LDAPS instead of LDAP (port 636)

```bash
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> --ldaps
```

**Force Kerberos Auth** - Force using Kerberos authentication instead of NTLM (when possible)

```bash
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> --force-kerb
```

**Verbose** - Enable all verbose and debug outputs

```bash
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> --verbose
```

**Interface** - Choose attacker's network interface

```bash
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> -I tun0
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> --interface eth0
```

**Targets** - Choose targets to be scanned (DC, All, IP=IP_or_hostname, File=./path_to_file)

```bash
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> --targets All
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> --targets DC
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> -T IP=192.168.0.1
./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain> -T File=./list_servers.txt
```

### Tunneling

linWinPwn can be particularly useful when you have access to an Active Directory environment for a limited time only, and you wish to be more efficient in the enumeration process and in the collection of evidence.
In addition, linWinPwn can replace the use of enumeration tools on Windows in the aim of reducing the number of created artifacts (e.g., PowerShell commands, Windows Events, created files on disk), and bypassing certain Anti-Virus or EDRs. This can be achieved by performing remote dynamic port forwarding through the creation of an SSH tunnel from the Windows host (e.g., VDI machine or workstation or laptop) to a remote Linux machine (e.g., Pentest laptop or VPS), and running linWinPwn with proxychains.

On the Windows host, run using PowerShell:
```powershell
ssh.exe kali@<linux_machine> -R 1080 -NCqf
```
On the Linux machine, first update `/etc/proxychains4.conf` to include `socks5 127.0.0.1 1080`, then run:
```bash
proxychains ./linWinPwn.sh -t <Domain_Controller_IP_or_Target_Domain>  -d <AD_domain> -u <AD_user> [-p <AD_password> -H <hash[LM:NT]> -K <kerbticket[./krb5cc_ticket]> -A <AES_key> -C <cert[./cert.pfx]>] [-o <output_dir>] [--auto]
```

### Interactive Mode Menus

Main menu
```
1) Re-run DNS Enumeration using adidnsdump
2) Active Directory Enumeration Menu
3) ADCS Enumeration Menu
4) Brute Force Attacks Menu
5) Kerberos Attacks Menu
6) SMB shares Enumeration Menu
7) Vulnerability Checks Menu
8) MSSQL Enumeration Menu
9) Password Dump Menu
10) AD Objects or Attributes Modification Menu
```

AD Enum menu
```
1) BloodHound Enumeration using all collection methods (Noisy!)
2) BloodHound Enumeration using DCOnly
3) ldapdomaindump LDAP Enumeration
4) enum4linux-ng LDAP-MS-RPC Enumeration
5) GPP Enumeration using netexec
6) MS-RPC Enumeration using netexec (Users, pass pol)
7) LDAP Enumeration using netexec (Users, passnotreq, userdesc, maq, ldap-checker, subnets)
8) Delegation Enumeration using findDelegation and netexec
9) bloodyAD All Enumeration
10) bloodyAD write rights Enumeration
11) SilentHound LDAP Enumeration
12) ldeep LDAP Enumeration
13) windapsearch LDAP Enumeration
14) LDAP Wordlist Harvester
15) Enumeration of RDWA servers
16) SCCM Enumeration using sccmhunter
17) LDAP Enumeration using LDAPPER
18) Adalanche Enumeration
19) GPO Enumeration using GPOwned
20) Open p0dalirius' LDAP Console
21) Open p0dalirius' LDAP Monitor
22) Open garrettfoster13's ACED console
23) Open LDAPPER custom options
```

ADCS menu
```
1) ADCS Enumeration using netexec
2) certi.py ADCS Enumeration
3) Certipy ADCS Enumeration
4) Certifried check
5) Certipy LDAP shell via Schannel (using Certificate Authentication)
```

BruteForce menu
```
1) RID Brute Force (Null session) using netexec
2) User Enumeration using kerbrute (Null session)
3) User=Pass check using kerbrute (Noisy!)
4) User=Pass check using netexec (Noisy!)
5) Pre2k computers authentication check (Noisy!)
```

Kerberos Attacks menu
```
1) AS REP Roasting Attack using GetNPUsers
2) Kerberoast Attack using GetUserSPNs
3) Cracking AS REP Roast hashes using john the ripper
4) Cracking Kerberoast hashes using john the ripper
5) NoPac check using netexec (only on DC)
6) MS14-068 check (only on DC)
7) CVE-2022-33679 exploit / AS-REP with RC4 session key (Null session)
8) AP-REQ hijack with DNS unsecure updates abuse using krbjack
9) Run custom Kerberoast attack using Orpheus
10) Generate Golden Ticket (requires: password or NTLM hash of Domain Admin)
11) Generate Silver Ticket (requires: password or NTLM hash of Domain Admin)
12) Generate Diamond Ticket (requires: password or NTLM hash of Domain Admin)
13) Generate Sapphire Ticket (requires: password or NTLM hash of Domain Admin)
```

SMB Shares menu
```
1) SMB shares Scan using smbmap
2) SMB shares Enumeration using netexec
3) SMB shares Spidering using netexec 
4) SMB shares Scan using FindUncommonShares
5) SMB shares Scan using manspider
```

Vuln Checks menu
```
1) zerologon check using netexec (only on DC)
2) MS17-010 check using netexec
3) PetitPotam check using netexec (only on DC)
4) dfscoerce check using netexec (only on DC)
5) Print Spooler check using netexec
6) Printnightmare check using netexec
7) WebDAV check using netexec
8) shadowcoerce check using netexec
9) SMB signing check using netexec
10) ntlmv1 check using netexec
11) runasppl check using netexec
12) RPC Dump and check for interesting protocols
13) Coercer RPC scan
```

MSSQL Enumeration menu
```
1) MSSQL Enumeration using netexec
2) MSSQL Relay check
```

Password Dump menu
```
1) LAPS Dump using netexec
2) gMSA Dump using netexec
3) DCSync using secretsdump (only on DC)
4) Dump SAM and LSA using secretsdump
5) Dump SAM and SYSTEM using reg
6) Dump NTDS using netexec
7) Dump SAM using netexec
8) Dump LSA secrets using netexec
9) Dump LSASS using lsassy
10) Dump LSASS using handlekatz
11) Dump LSASS using procdump
12) Dump LSASS using nanodump
13) Dump LSASS using masky (ADCS required)
14) Dump dpapi secrets using netexec
15) Dump secrets using DonPAPI
16) Dump NTDS using certsync (ADCS required) (only on DC)
17) Dump secrets using hekatomb (only on DC)
18) Search for juicy credentials (Firefox, KeePass, Rdcman, Teams, WiFi, WinScp)
19) Dump Veeam credentials (only from Veeam server)
20) Dump Msol password (only from Azure AD-Connect server)
21) Extract Bitlocker Keys
22) Privilege escalation from Child Domain to Parent Domain using raiseChild
```

Modification menu
```
1) Targeted Kerberoast Attack (Noisy!)
2) Change user or computer password (Requires: ForceChangePassword on user or computer)
3) Add user to group (Requires: GenericWrite or GenericAll on group)
4) Add new computer (Requires: MAQ > 0)
5) Perform RBCD attack (Requires: GenericWrite or GenericAll on computer)
6) Perform ShadowCredentials attack (Requires: AddKeyCredentialLink)
7) Abuse GPO to execute command (Requires: GenericWrite or GenericAll on GPO)
```

Auth menu
```
1) Generate and use NTLM hash of current user (requires: password) - Pass the hash
2) Crack NTLM hash of current user and use password (requires: NTLM hash)
3) Generate and use TGT for current user (requires: password, NTLM hash or AES key) - Pass the key/Overpass the hash
4) Extract NTLM hash from Certificate using PKINIT (requires: pfx certificate)
5) Request and use certificate (requires: authentication)
```

Config menu
```
1) Check installation of tools and dependencies
2) Synchronize time with Domain Controller (requires root)
3) Add Domain Controller's IP and Domain to /etc/hosts (requires root)
4) Update resolv.conf to define Domain Controller as DNS server (requires root)
5) Update krb5.conf to define realm and KDC for Kerberos (requires root)
6) Download default username and password wordlists (non-kali machines)
7) Change users wordlist file
8) Change passwords wordlist file
9) Change attacker's IP
10) Switch between LDAP (port 389) and LDAPS (port 636)
11) Show session information
```

## Demos
- HackTheBox Forest

Interactive Mode:
[![asciicast](https://asciinema.org/a/499893.svg)](https://asciinema.org/a/499893)

Automated Mode:
[![asciicast](https://asciinema.org/a/464904.svg)](https://asciinema.org/a/464904)

- TryHackme AttacktiveDirectory

[![asciicast](https://asciinema.org/a/464901.svg)](https://asciinema.org/a/464901)

## TO DO

- Add more enumeration and exploitation tools...

## Credits

- Inspiration: [S3cur3Th1sSh1t](https://github.com/S3cur3Th1sSh1t) - WinPwn
- Tools: 
    - [fortra](https://github.com/fortra) - impacket
    - [NeffIsBack, Marshall-Hallenbeck, zblurx, mpgn, byt3bl33d3r and all contributors](https://github.com/Pennyw0rth/NetExec) - crackmapexec/netexec
    - [Fox-IT](https://github.com/fox-it) - bloodhound-python
    - [dirkjanm](https://github.com/dirkjanm/) - ldapdomaindump, adidnsdump
    - [zer1t0](https://github.com/zer1t0) - certi.py
    - [ly4k](https://github.com/ly4k) - Certipy
    - [ShawnDEvans](https://github.com/ShawnDEvans) - smbmap
    - [ropnop](https://github.com/ropnop) - windapsearch, kerbrute
    - [login-securite](https://github.com/login-securite) - DonPAPI
    - [Processus-Thief](https://github.com/Processus-Thief) - HEKATOMB
    - [layer8secure](https://github.com/layer8secure) - SilentHound
    - [ShutdownRepo](https://github.com/ShutdownRepo) - TargetedKerberoast
    - [franc-pentest](https://github.com/franc-pentest) - ldeep
    - [garrettfoster13](https://github.com/garrettfoster13/) - pre2k, aced, sccmhunter
    - [zblurx](https://github.com/zblurx/) - certsync
    - [p0dalirius](https://github.com/p0dalirius) - Coercer, FindUncommonShares, ExtractBitlockerKeys, LDAPWordlistHarvester, ldapconsole, pyLDAPmonitor, RDWAtool
    - [blacklanternsecurity](https://github.com/blacklanternsecurity/) - MANSPIDER
    - [CravateRouge](https://github.com/CravateRouge) - bloodyAD
    - [shellster](https://github.com/shellster) - LDAPPER
    - [TrustedSec](https://github.com/trustedsec) - orpheus
    - [lkarlslund](https://github.com/lkarlslund) - Adalanche
    - [X-C3LL](https://github.com/X-C3LL) - GPOwned
    - [Hackndo](https://github.com/Hackndo) - pyGPOAbuse
    - [CompassSecurity](https://github.com/CompassSecurity) - mssqlrelay

- References:
    -  https://orange-cyberdefense.github.io/ocd-mindmaps/
    -  https://github.com/swisskyrepo/PayloadsAllTheThings
    -  https://book.hacktricks.xyz/
    -  https://adsecurity.org/
    -  https://casvancooten.com/
    -  https://www.thehacker.recipes/
    -  https://www.ired.team/
    -  https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet
    -  https://hideandsec.sh/

## Legal Disclamer

Usage of linWinPwn for attacking targets without prior mutual consent is illegal. It's the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program. Only use for educational purposes.
