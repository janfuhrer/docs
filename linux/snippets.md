# docs: linux/snippets
#linux #snippets 

## links
- ExplainShell: https://explainshell.com/
- Showthedocs: http://showthedocs.com
- Shell Parameter Expansion:
	- https://www.gnu.org/software/bash/manual/bash.html#Shell-Parameter-Expansion
	- https://wiki-dev.bash-hackers.org/syntax/pe
- Regex: https://regex101.com
- YAML multiline: https://yaml-multiline.info

## openssl
#openssl

```bash
# pkcs12
openssl pkcs12 -nokeys -info -in cert.p12 -passin pass:${password}

# x509
openssl x509 -in cert.crt -text -noout

# der to base64
openssl x509 -inform der -in cert-der.crt -out cert-base64.crt
```

## nc
- `-v`: verbose
- `-z`: just scan for listening daemons, without sending any data to them

```bash
nc -zv ${ip} ${port}
```

### send udp packages between hosts
- client 1
```bash
nc -u -v -l ${ip} 2000
```
- client 2
```bash
nc -u ${ip} 2000
```

## grep
**find pattern in directory-structure**
- `-r`: recursived
- `n`: line number
- `w`: math the whole word

```bash
grep -rnw '${path}' -e 'pattern'
```

## send specifig packages
install
```bash
apt install nmap
```

start listening on remote side
```bash
# listen for ICMP type 3 packets
sudo tcpdump  'icmp[0] = 3' -i ${interface}
sudo tcpdump  'icmp[0] = 3 and (host ${source-ip})' -i ${interface}
```

send on the other side
```bash
# send ICMP type 3 packets
sudo nping --icmp-type 3 ${remote-ip}

# send tcp packages with data-length 1000
sudo nping --tcp -p 22 --data-length 1000 ${remote-ip}
```

## vim
#vim

Link: https://vim.rtorr.com

- `:set expandtab`: insert space when tab key is pressed
- `:set tabstop=4`: number of space characters when tab key is pressed
- `:retab`: change all existing tab-characters to the current setting
- `:set shiftwidth=4`: number of space characters inserted for indentation

Example: `set expandtab tabstop=2 shiftwidth=2`

**Indent over multiple lines**
- change to visual mode and mark all the lines
- press `>>` to indent or `<<` de-indent

## sed
- with macOS, a backup file must be specified with `-i` -> to overwrite the file directly use `-i''` 

```bash
sed -i '' 's/old/new/g' ${filename}
```

## various
### password
> If a User has specific characters in his password (like `$`), the password can't be set with the command `passwd`. In this instruction, the password-hash is manually generate and set in the file `/etc/shadow`.

Create user:
```bash
useradd -g ${gid} -u ${uid} -c "${comment}" -d ${home-directory} -s /bin/bash ${username}
```

Generate Password-Hash (`$` must be set!):
```python
python
>>> import crypt
>>> crypt.crypt('<PASSWORD>', '$<ALGORITHM>$<SALT>')
>>> quit() 
```

**\<ALGORITHM\>**
Only 1, 2a, 2y, 5 or 6 possible. Preffered algorithm is 6 (SHA512)
- `1`: MD5
- `2a`: Blowfish
- `2y`: Blowfish, with correct handling of 8 bit characters
- `5`: SHA256
- `6`: SHA512

**\<SALT\>**
Random string (f.e. 8 characters)

Put new Hash in `/etc/shadow` (if available in `/etc/shadow-` as well):
```bash
vim /etc/shadow
vim /etc/shadow-
```