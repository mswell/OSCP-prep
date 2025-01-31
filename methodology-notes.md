
https://paper.dropbox.com/doc/OSCP-Methodology-EnVX7VSiNGZ2K2QxCZD7Q

# WEB
# Step 1:Basic scan
~~~
TCP SCAN 
nmap -sC -sV -oA -ip-
Nmap -Pn -p- -vv -ip-
UDP SCAN
Nmap -Pn -p- -sU -vv -ip-
~~~
# Step 2: Nmap version and vulnerability Scan: 
~~~
https://nmap.org/nsedoc/ -- scripts
Nmap -Pn -sV -O -pT:{TCP ports found in step 1},U:{UDP ports found in step 1} -script *vuln* <ip address>
Grab banners manually for more clarity: nc -nv <ip-address> <port>
 ~~~ 
# Automatic scanners + Vulnerability Assessments tools 
- Kaboom https://github.com/Leviathan36/kaboom 
- Trigmap https://4hou.win/wordpress/?p=32665 
 
# Step 2.1: Subdomains check 
~~~
knock # python knockpy.py target.com
massscan 
sublist3r # python3 sublist3r.py -v -d target.com

alternative 
https://github.com/OWASP/Amass/ 
~~~
# Step 3: Any web dirs/sensitive data on subs/domain?:
~~~
Nikto -port {web ports} -host ip address -o output file.txt
Dirb http{s}://ip address:port /usr/share/wordlist/dirb/{common/small/vulns}.txt
Gobuster -u http://<ip-address> -w /usr/share/Seclists/Discovery/Web_Content/common.txt
if no dirs/contect found - test for customized dirs based on target name, clues - etc.
/usr/share/secLists/Discovery folder has some great word lists
If no web dirs visible try a bigger list in dirb: /usr/share/wordlist/dirb/big.txt
if no dirs then check for customized dirs for each sub based on target keywords, info
Use Burpsuite if needed
Do you see any interesting directory containing sensitive data?
Do you see any LFI/RFI vulnerability posted by Nikto? Try fimap: fimap -u <ip-address>
~~~
# Step 4: Are there any exploits available publicly from the services discovered from Step 2?
~~~
Searchsploit <service name>
~~~
# Step 5: Manual tests for Web pages/app
~~~
Check the Page Source, Inspect elements, view cookies, tamper data, use curl/wget

    Google alien terms!
    Anything sensitive there?
    Any version info?
    Headers?
    Dirs or syntax crawlers cannot catch?
~~~
Search object online (like GitHub) if the application used is open source: this may assist in site enumeration and guessing versions etc.!

# part of step5 > Check HTTP Options

# Check for Input Validation in forms 
~~~
(like: 1′ or 1=1 limit 1;#   AND   1′ or 1=1–)
    NULL or null
        Possible error messages returned.
    ‘ , ” , ; , <!
        Breaks an SQL string or query; used for SQL, XPath and XML Injection tests.
    – , = , + , ”
        Used to craft SQL Injection queries.
    ‘ , &, ! , ¦ , < , >
        Used to find command execution vulnerabilities.
    ../
        Directory Traversal Vulnerabilities. 
                                     
       Check network during page load > any additional files / does it take source from remote addr / pull data / etc
~~~                                   
# NETWORK
                                     
# Step 5.1:Semi auto recon + exploit suggest based on results - https://github.com/frizb/Vanquish                                      
# Step 6: Are there any NETBIOS, SMB, RPC ports discovered from Step 1?
~~~
enum4linux -a <ip address>
~~~
~~~
Rpcclient <ip address> -U “” -N
~~~
~~~
Rpcinfo: What services are running? Rpcinfo -p <target ip>
~~~
~~~
Is portmapper running? Is rlogin running? Or NFS or Mountd?

http://etutorials.org/Networking/network+security+assessment/Chapter+12.+Assessing+Unix+RPC+Services/12.2+RPC+Service+Vulnerabilities/

Showmount -e <ip address>/<port>

Can you mount the smb share locally?

Mount -t cifs //<server ip>/<share> <local dir> -o username=”guest”,password=””

Rlogin <ip-address>

Smbclient -L \\<ip-address> -U “” -N

Nbtscan -r <ip address>

Net use \\<ip-address>\$Share “” /u:””

Net view \\<ip-address>

Check NMAP Scripts for SMB, DCERPC and NETBIOS

# Step 7: Any SMTP ports available?

Enumerate Users:

Mail Server Testing

    Enumerate users
        VRFY username (verifies if username exists – enumeration of accounts)
        EXPN username (verifies if username is valid – enumeration of accounts)

# Step 8: How about SNMP ports?
~~~
~~~
Default Community Names: public, private, cisco, manager

Enumerate MIB:

1.3.6.1.2.1.25.1.6.0 System Processes

1.3.6.1.2.1.25.4.2.1.2 Running Programs

1.3.6.1.2.1.25.4.2.1.4 Processes Path

1.3.6.1.2.1.25.2.3.1.4 Storage Units

1.3.6.1.2.1.25.6.3.1.2 Software Name

1.3.6.1.4.1.77.1.2.25 User Accounts

1.3.6.1.2.1.6.13.1.3 TCP Local Ports

Use tools:

Onesixtyone – c <community list file> -I <ip-address>

Snmpwalk -c <community string> -v<version> <ip address>

Eg: enumerating running processes:

root@kali:~# snmpwalk -c public -v1 192.168.11.204 1.3.6.1.2.1.25.4.2.1.2
~~~

# Step 9: FTP Ports Discovered
~~~
Is anonymous login allowed?

If yes, is directory listing possible? Can a file be ‘get’ or ‘send’?

Use browser: ftp://<ip-address> , What do you find?
~~~

# Step 10: Password Cracking / Brute Forcing
~~~

Try this as the last resort or in case the Passwd/Shadow/SAM files are in possession:

For linux, first combine passwd & shadow files:  unshadow [passwd-file] [shadow-file] > unshadowed.txt

Then, use John on the unshadowed file using a wordlist or rules mangling : john –rules –wordlist=<wordlist file> unshadowed.txt

Identifying Hash: hash-identifier

For other services, use Medusa or Hydra. Eg:

Hydra -L <username file> -P <Password file> -v <ip-address> ssh

Medusa -h <ip-address> -U <username file> -P <password file> -M http -m DIR:/admin -T 30

Using hashcat for cracking hashes:

For WordPress MD5 with salt: hashcat -m 400 -a 0 <hash file> <wordlist file>

Sample Password list: /usr/share/wordlist/rockyou.txt
~~~

# Step 11: Packet Sniffing
~~~

Use Wireshark / tcpdump to capture traffic on the target host:

“tcpdump -i tap0 host <target-ip> tcp port 80 and not arp and not icmp -vv”
~~~
