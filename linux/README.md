# docs: linux
## snippets
### openssl
```bash
# pkcs§1
openssl pkcs12 -nokeys -info -in cert.p12 -passin pass:PASS

# x509
openssl x509 -in cert.crt -text -noout

# der to base64
openssl x509 -inform der -in CACert.crt -out intermediate.cer
```

### nc
```bash
nc -zv 192.168.1.1 22
```

### grep
**find pattern in directory-structure**
`-r`: recursived
`n`: line number
`w`: math the whole word

```bash
grep -rnw '/path/to/somewhere/' -e 'pattern'
```

### vim
Link: https://vim.rtorr.com

- `:set expandtab`: insert space when tab key is pressed
- `:set tabstop=4`: number of space characters when tab key is pressed
- `:retab`: change all existing tab-characters to the current setting
- `:set shiftwidth=4`: number of space characters inserted for indentation

Example: `set expandtab tabstop=2 shiftwidth=2`

**Indent over multiple lines**
- change to visual mode and mark all the lines
- press `>>` to indent or `<<` de-indent

### sed
- with macOS, a backup file must be specified with `-i` -> to overwrite the file directly use `-i''` 

```bash
sed -i '' 's/old/new/g' filename
```

## various
### nextcloud
turn maintenance mode off

```bash
su -s /bin/bash www-data -c "php occ maintenance:mode --off"
```

### password
> If a User has specific characters in his password (like `$`), the password can't be set with the command `passwd`. In this instruction, the password-hash is manually generate and set in the file `/etc/shadow`.

Create user:
```bash
useradd -g <GID> -u <UID> -c "<COMMENT>" -d <HOME-DIRECTORY> -s /bin/bash <USERNAME>
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