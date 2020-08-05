# Ninja Skills Write-up
> (incomplete)
---

> one liner commands to solve the problems.

### Tasks

```
Answer the questions about the following files:

8V2L
bny0
c4ZX
D8B3
FHl1
oiMO
PFbD
rmfX
SRSq
uqyw
v2Vb
X1Uy

The aim is to answer the questions as efficiently as possible.
```
	
Which of the above files are owned by the best-group group(enter the answer separated by spaces in alphabetical order)
```
find / -type f -user new-user -group best-group -exec ls -ldb {} \; 2>/dev/null
``` 
> Answer: D8B3 v2Vb

Which of these files contain an IP address?
```
find / -type f -user new-user -size 13545c -exec ls -ldb {} \; 2>/dev/null | awk -F" " '{print $9}' | xargs grep -o '[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}'
```
> Answer: oiMO

Which file has the SHA1 hash of 9d54da7584015647ba052173b84d45e8007eba94
```
find / -type f -user new-user -size 13545c -exec ls -ldb {} \; 2>/dev/null | cut -d' ' -f10 | xargs sha1sum | grep 9d54da7584015647ba052173b84d45e8007eba94
```
> Answer: c4ZX

	
Which file contains 230 lines?
```
find / -type f -user new-user -exec ls -ldb {} \; 2>/dev/null | awk -F" " '{print $9}' | xargs wc -l 2>/dev/null | grep '230'
```
> Answer: Nothing appeared. It's not on the server as well. One filename didn't appear. "bny0" is the answer

Which file's owner has ID of 502?
```
find / -type f -user 502 2>/dev/null | xargs echo | cut -d' ' -f2
```
> Answer: X1Uy

Which file is executable by everyone?
```
find / -type f -user new-user -executable 2>/dev/null
```
> Answer: 8V2L
