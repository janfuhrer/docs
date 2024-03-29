tags: #linux #snippets

# linux-snippets

links: [[100 Linux MOC|Linux MOC]] - [[000 Index|Index]]

---

## links
- ExplainShell: https://explainshell.com/
- Showthedocs: http://showthedocs.com
- Shell Parameter Expansion:
	- https://www.gnu.org/software/bash/manual/bash.html#Shell-Parameter-Expansion
	- https://wiki-dev.bash-hackers.org/syntax/pe
- Regex: https://regex101.com
- YAML multiline: https://yaml-multiline.info
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

- delete last 4 lines of every file in this directory (and subdirectory), wich contains "search-string"
```bash
grep -iRl "search-string" . | xargs -I {} sh -c "cat {} | tac | sed '1,4 d' | tac > {}.test && mv {}.test {}"
```

- rename every file `README.md.gotpl` to `README.md.gotmpl`
```bash
for f in ./**/README.md.gotpl; do mv "$f" "$(echo "$f" | sed s/gotpl/gotmpl/)"; done
```

## hosts
- lookup via hosts-file
```bash
getent hosts $(hostname)
```

## test port access

```bash
python3 -m http.server --bind 0.0.0.0 8080
```

## extract deb file
- extract to the current directory
```bash
dpkg-deb -xv ${file.deb} .
```

## conda

```bash
# create env
conda create --name ${ENV}

# get envs
conda info --envs

# activate env
conda activate ${ENV}

# install pip 
conda install -c anaconda pip

# upgrade pip
python3.12 -m pip install --upgrade pip

# upgrade all packages
pip --disable-pip-version-check list --outdated --format=json | python -c "import json, sys; print('\n'.join([x['name'] for x in json.load(sys.stdin)]))" | xargs -n1 pip install -U

# install a python package in environment
pip install flask

# remove environment
onda remove -n ${ENV} --all
```

**Update python version**

```bash
conda activate ${ENV}

# update to python 3.12
conda install python=3.12

# install module
python3.12 -m pip show markupsafe

# check module
python3.12 -m pip show markupsafe

# check that ansible use correct python version
ansible --version
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

---
links: [[100 Linux MOC|Linux MOC]] - [[000 Index|Index]]