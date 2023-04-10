# docs: macOS/snippets
#macOS #snippets 

## some useful shortcuts
```bash
## git: https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git
ga -p # git add -p
gc! # git commit -v -a --amend
gcsm # git commit -s -m
gcb # git checkout -b
gcs # git commit -S
gd # git diff
gds # git diff --staged
gf # git fetch
gpsup # git push --set-upstream origin $(git_current_branch)
gp # git push
gst # git status

## kubectl: https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/kubectl
kgp # kubectl get pods
kgs # kubectl get service
kl # kubectl get logs -> klf -f

## macos: https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/macos
tab # open curent directory in a new tab
spotify # control spotify
rmdsstore # Remove .DS_Store files recursively in a directory
quick-look # Quick-Look a specified file
ofd # Open the current directory in a Finder window
```

## oh my zsh
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

## vim .zshrc_oh-my-zsh
plugins=(ssh-agent git compleat zsh-syntax-highlighting zsh-autosuggestions kubectl gpg-agent golang macos)
```

## remove quarantine attributes
```bash
## single file
xattr -d com.apple.quarantine ${FILE}

## directory
xattr -r -d com.apple.quarantine ${DIRECTORY}
```

## problems with external HD
**Problem**
- APFS on external HD is corrupt, spaceManager gives error, HD not mountable
- erase disk with DiskUtility not possible

**Solution**
1. erase disk via Terminal

```bash
diskutil eraseDisk JHFS+ CleanDrive /dev/disk2
```

2.  erase Disk in DiskUtil Manager and reinit TimeMachine

## dns
```bash
scutil --dns
sudo killall -HUP mDNSResponder
```

## backup & restore RaspberryPi
Get Disk-Number:
```bash
diskutil list
```

### backup SD-Card
1. Save SD-Card to Image (f.e. disk3 to rpi-image.gz):
```bash
sudo dd bs=4m if=/dev/disk3 | gzip > ~/Desktop/rpi-image.gz
```
Um den Status des Kopiervorgangs anzuzeigen, kann die folgende Tastenkombination verwendet werden: CTRL + T

### restore SD-Card
1. Umount disk:
```bash
diskutil umountDisk /dev/disk3
```
2. SD-Card in FAT formatieren.
```bash
sudo newfs_msdos -F 16 /dev/disk3
```

3. Restore Image to SD-Card:
```bash
sudo gzip -dc ~/Desktop/rpi-image.gz | sudo dd bs=4m of=/dev/disk3
```
Um den Status des Kopiervorgangs anzuzeigen, kann die folgende Tastenkombination verwendet werden: CTRL + T

## install & configs
### tmux
```bash
vim ~/.tmux.conf
vim ~/.battery.zsh
```
### vim
```bash
vim ~/.vimrc
```

### EJSON
Github: https://github.com/Shopify/ejson

install
```bash
brew tap shopify/shopify
brew install ejson
```

```bash
sudo mkdir -p /opt/ejson/keys
sudo chown -R $(whoami) /opt/ejson
```

### Wireshark

```bash
brew install --cask wireshark

# for ksniff
echo '#!/bin/sh
exec /Applications/Wireshark.app/Contents/MacOS/Wireshark "$@"' > /usr/local/bin/wireshark

chmod +x /usr/local/bin/wireshark
```