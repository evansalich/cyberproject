# Vulnerability Assessment Report - vsftpd Backdoor Exploitation
This document outlines the setup and execution of a demonstration showcasing a rudimentary example of a Backdoor Exploitation.

## Attack Summary
A penetration test was conducted against a target system running `vsftpd 2.3.4` to assess the impact of the known backdoor vulnerability (CVE-2011-2523). 
The assessment successfully demonstrated that an unauthenticated attacker can gain root-level access to the target system through the FTP service.

## Threat Model
The necessary Threat Model for this attack entails the adversary possessing the following capabilities:
* The attacker has access on the network to comunicate with the vulnerable system
* The targeted machine has the 2.3.4 version of the vsftpd

## Demo Setup
To execute the Demo described below, we'll need two Virtual Machines configured as follows
* The attacker side has a VM loaded with `Kali` Linux with `IPv4 192.168.1.4` and the `rockyou.txt` wordlist already decompressed
* The defender side has a VM loaded with `Metasploitable 2` with `IPv4 192.168.1.3`

Both machine are in the same network (`192.168.1.0/24`) in order to comunicate with each other. 

(foto di metasploitable 2)

## Attack Flow

### Network Discovery 
The first attack step is to scan the entire subnetwork in order to identify the target system and confirming the presence of the vulnerable service. 
An nmap scan was used to find the defender IP and TCP ports open scanning the entire `/24` subnetwork, in the demo the TCP ports where already shown after the command but sometimes it is necessary to launch another nmap command specifing the right IP address of the defending system.

(foto nmap)

### Backdoor Exploitation
A connection was established to the FTP service (running on port 21) using netcat. The goal was to interact with the vsftpd server and deliver the malicious payload that would trigger the backdoor. 

Upon connection, the FTP banner was received, confirming that the service was responsive and ready to accept commands. The backdoor trigger was delivered by sending a specially crafted username to the FTP server. The vsftpd 2.3.4 backdoor activates when a username that ends by the sequence `:)` is submitted. While the username has a crucial part, the password field can contain any arbitrary value.

(foto exploit)

When the FTP server processes the username containing `:)`, the malicious code embedded in the vsftpd binary executes. This code binds a root-level command shell to TCP port 6200 on the target system, bypassing any authentication requirements.

A verification scan was performed by using the nmap command with the option `-sS` that will check if the port 6200 is opened but it won't connect to it, avoiding so the risk of closing it.

(foto nmap -sS)

A netcat connection was established to port 6200, providing to the attacker direct access to a root shell on the target system. After conferming that root priviledge are aquired an cat /etc/shadow command was performed in order to extract the password hashes from the system.

The /etc/shadow file was copied and saved for offline password cracking. The following user accounts with crackable hashes were identified: 

| Username | Hash Type | Hash Value                           |
| -------- | --------- | ------------------------------------ |
| root     | MD5-crypt | `$1$/avpfBJ1$x0z8w5UF9Iv./DR9E9Lid.` |
| sys      | MD5-crypt | `$1$fUX6BP0t$Miyc3Up0zQJqz4s5wFD9l0` |
| klog     | MD5-crypt | `$1$fZ2VMS4K$R9XK1.CmLdHhduE3X9jqP0` |
| msfadmin | MD5-crypt | `$1$XN10ZJ2c$Rt/zZcW3mLtUWA.iHzjA5/` |
| postgres | MD5-crypt | `$1$Rw35ik.x$MgQqZUu05pAoUvfJhfCYe/` |
| user     | MD5-crypt | `$1$HESu9xrH$k.o3G93DGoXIIqKKPmUgZ0` |
| service  | MD5-crypt | `$1$kR3ue7JZ$7GxELDupr50hp6cjZ3Bu/`  |

### Password Cracking
The extracted hashes were saved to a file and processed using John the Ripper. Multiple attack modes were employed to maximize success. 

The first mode used was to compare those hashes with the rockyou.txt wordlist already present in Kali. 

(foto 1st try con john)

Not all the hashes were present on the selected wordlist so only three out of seven hashes where successfuly cracked. So we had to try to crack them with other modes, we had to use the option `--single` that was able to crack three other hashes.

(foto 2nd try)

The only left out hash was the one related to root that has a password either not-trivial or not present on the wordlist and so we weren't able to crack it, so the final result of the attack was the following hashes

(foto final)

## Conclusions 
This penetration test successfully demonstrated the critical severity of the vsftpd 2.3.4 backdoor vulnerability (CVE-2011-2523) against the Metasploitable 2 target. The attack was executed with minimal effort and no authentication, resulting in complete compromise of the target system. 

For the Metasploitable 2 environment specifically, the system is intentionally vulnerable for educational purposes. However, this assessment demonstrates real-world attack patterns that defenders must be prepared to detect and block.

## Credits
- **Video Reference:** Metasploit Hacking Demo (includes password cracking) by David Bombal https://www.youtube.com/watch?v=bBut8D7usKA&list=PL-9Wo53rJTQrWYLYaBvZA14FbE-GaFKjO)
- **Technical Guidance:** DeepSeek AI https://chat.deepseek.com
- **Markdown Template:** John Gruber found in Docsify-This https://daringfireball.net/projects/markdown/

The LLM provided technical guidance, command assistance, and report revision. 



### Images
Images have a similar syntax to links but include a preceding exclamation point.

```markdown
![Image of Minion](https://octodex.github.com/images/minion.png)
```
![Image of Minion](https://octodex.github.com/images/minion.png)

and using a local image (which also displays in GitHub):

```markdown
![Image of Octocat](images/octocat.jpg)
```
![Image of Octocat](images/octocat.jpg)
