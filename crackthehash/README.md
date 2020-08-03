# Crack the Hash Write-Up
> Earn the hash cracker badge upon completion
---

Note: Rough writeup. Posting commands and results.

---

### Hash Identifier
> parrot / kali os
```
root@kali# hash-identifier
root@kali# hashid -h
```

### Task 1

MD5 => 8bb6e862e54f2a795ffc4e541caed4d
```
hashcat -a 0 -m 0 md5.hash /usr/share/wordlists/rockyou.txt --force -O
```

SHA1 => CBFDAC6008F9CAB4083784CBD1874F76618D2A97
```
hashcat -a 0 -m 100 sha1.hash /usr/share/wordlists/rockyou.txt --force -O
```

SHA256 => 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
```
hashcat -a 0 -m 1400 sha256.hash /usr/share/wordlists/rockyou.txt --force -O
```
	
bcrypt => $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
```
hashcat -a 0 -m 3200 bcrypt.hash /usr/share/wordlists/rockyou.txt --force -O
```

MD4 => 279412f945939ba78ce0758d3fd83daa
```
hashcat -a 0 -m 900 md4.hash /usr/share/wordlists/rockyou.txt --force -O
```

---

### Task 2

SHA256 => F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
```
hashcat -a 0 -m 1400 sha256.hash /usr/share/wordlists/rockyou.txt --force -O
```

NTLM => 1DFECA0C002AE40B8619ECF94819CC1B
```
hashcat -a 0 -m 1000 ntlm.hash /usr/share/wordlists/rockyou.txt --force -O
```
sha512crypt (UNIX) (5rounds) => $6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02. 
```
hashcat -a 0 -m 1800 sha512crypt.hash /usr/share/wordlists/rockyou.txt --force -O
```

HMAC-SHA1 => e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme
```
hashcat -a 0 -m 160 hmac-sha1.hash /usr/share/wordlists/rockyou.txt --force -O
```
