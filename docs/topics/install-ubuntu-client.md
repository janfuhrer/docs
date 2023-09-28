tags: #linux #ubuntu #client

# install-ubuntu-client

links: [[100 Linux MOC|Linux MOC]] - [[000 Index|Index]]

---

Quelle: https://askubuntu.com/questions/918021/encrypted-custom-install

1. Erstelle einen Live Boot-Stick mit Ubuntu: https://ubuntu.com/download/desktop
2. Als nächstes wird die Festplatte für die Installation vorbereitet. Dazu starten wir via USB-Stick das Live-System auf dem neuen Notebook.

> Das Betriebssystem wird erst später installiert, wir starten **nur** die Live-Umgebung!

### Partitionierung mit gparted
Mit dem grafischen Tool gparted wird die Festplatte wie folgt formatiert:

- **EFI-Partition**: sda1 - 512MB fat32
- **Boot-Partition**: sda2 - 2048MB ext4
- **Main-Partition**: sda3 - 474.4GB unformatted


### Verschlüsselung mit LUKS
Für die Verschlüsselung des Betriebssystems und den Benutzerdaten verschlüsseln wir die "Main-Partition" /dev/sda3.

```bash
sudo cryptsetup luksFormat --hash=sha512 --verify-passphrase /dev/sda3
sudo cryptsetup luksDump /dev/sda3
```
Nach dem zweitem Befehl erhalten wir die UUID der LUKS-Partition. Hier wird der erste Block (in unserem Beispiel decc495a) für den Namen der LUKS-Partition verwendet. 

```bash
sudo cryptsetup luksOpen /dev/sda3 luks-decc495a
```

### Konfiguration LVM
Nachdem die Festplatte verschlüsselt wurde, konfigurieren wir die logischen Volumes mit LVM (Logical Volume Manager). Wir erstellen folgendes Schema:

- root: 50GB
- home: 150GB
- swap: 8GB

```bash
sudo pvcreate /dev/mapper/luks-decc495a
sudo vgcreate vg0 /dev/mapper/luks-decc495a
sudo lvcreate -n root -L 50G vg0
sudo lvcreate -n home -L 150G vg0
sudo lvcreate -n swap -L 8G vg0
```

## Installation Ubuntu
Nun kann der Installations-Wizard gestartet werden und die Installation durchgeführt werden. Das Speicherschema wird wie folgt konfiguriert:

- sda1: /boot/efi
- sda2: /boot
- sda3:
  - vg0-root: /
  - vg0-home: /home
  - vg0-swap: [swap]
  
> Nach der Installation den Computer **nicht** neustarten!
 
### Automount LUKS-Partition
Bevor der Computer neugestartet werden kann, müssen einige Konfigurationen für die automatische Einhängung der LUKS-Partition durchgeführt werden.
Zuerst wird die **UUID** der Partition notiert.
```bash
sudo blkid | grep LUKS
```
Nun hängen wir einige Partitionen ein.
 ```bash
sudo mount /dev/vg0/root /mnt
sudo mount /dev/vg0/home /mnt/home
sudo mount /dev/sda2 /mnt/boot
sudo mount --bind /dev /mnt/dev
sudo mount --bind /run/lvm /mnt/run/lvm
sudo mount /dev/sda1 /mnt/boot/efi
```
Erstellen eine chroot-Umgebung.
 ```bash
sudo chroot /mnt
mount -t proc proc /proc
mount -t sysfs sys /sys
mount -t devpts devpts /dev/pts
```
Die Partition wird nun in der Datei `/etc/crypttab` erfasst.
```bash
sudo vim /etc/crypttab
```
Die Zeiele sollte wie folgt aussehen (UUID und Namen anpassen!).
`luks-decc495a UUID=decc495a-9f68-4072-b255-a4f6dc030457 none luks,discard`

Nun folgen zwei weitere Befehle.
```bash
update-initramfs -k all -c
update-grub
```

Jetzt kann der Computer neu gestartet werden und der USB-Stick entfernt werden.

## Konfiguration System
### Zweiter LUKSKeySlot
```bash
sudo cryptsetup luksAddKey /dev/sda3
```

### Installation Gnome Desktop
```bash
sudo apt-get install vanilla-gnome-desktop
```

### Installation und Konfiguration Firewall
```bash
sudo apt-get install ufw gufw

sudo ufw enable
sudo ufw default deny incoming
sudo ufw reload
sudo ufw status verbose
```

### Installation Utilities
**Diverse Programme**
```bash
sudo apt-get install snap vim keepassxc evolution evolution-ews tmux
```

### Anpassung Design
**Extensions**
Download via Softwarecenter:
- Dash to Dock
- TopIcons
- Caffeine
- Blyr
- Transparent GNOME panel

**Themes**
1. Download von: https://gnome-look.org
  - ANT-Theme (`mv Ant ~/.local/share/themes/`)
  - El-Capitan-Cursors (`mv El_Capitan_Cursors ~/.local/share/icons/`)
  - Os-Catalania-icons (`mv Os-Catalania-icons ~/.local/share/icons/`)
2. Aktiviere Theme mit dem Programm **Tweaks**

**Shortcut Ctrl + Return im Nautilus anpassen**
Damit im Filebrowser Nautilus Dateien mit dem Shortcut Ctrl + Return unbenannt werden können, muss das folgende Script erstellt werden:
```bash
sudo apt-get install python-nautilus
mkdir -p ~/.local/share/nautilus-python/extensions
vim ~/.local/share/nautilus-python/extensions/BackspaceBack.py
nautilus -q
```


### Editorconf
Install Plugin [Editorconfig](https://editorconfig.org)

1. Download Plugin for vim from Github [Link](https://github.com/editorconfig/editorconfig-vim#readme)
2. Extract the archive and move the folders `autoload`, `doc`and `plugin` to `~/.vim`
```bash
https://github.com/editorconfig/editorconfig-vim/archive/master.zip
unzip ~/Downloads/editorconfig-vim-master.zip
mv ~/Downloads/editorconfig-vim-master/autoload ~/.vim/
mv ~/Downloads/editorconfig-vim-master/doc ~/.vim/
mv ~/Downloads/editorconfig-vim-master/plugin ~/.vim/
```
3. Create the file `.editorconfig` on the right place
```bash
vim ~/home/workspace/.editorconfig
```

4. Sample-File
```bash
# http://editorconfig.org
root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.md]
trim_trailing_whitespace = false
```

### Konfiguration vim
1. Edit `vimrc` 
```bash
vim ~/.vimrc
```
2. Sample-File
```bash
syntax on                                                                               
set background=dark                                                                     
set autoindent                                                                           
set cindent
```

## OpenVPN
1. Pakete installieren:
```bash
sudo apt-get install nfs-common cifs-utils libnss-myhostname opnvpn
```

2. Neue Aliase in `~/.bashrc` definieren:
```bash
cat <<EOF >> ~/.bashrc
alias vpn='sudo openvpn --config /etc/openvpn/config.ovpn'
alias vpnadd='/etc/openvpn/addproxy.sh'
alias vpnrem='/etc/openvpn/remproxy.sh'
EOF
```

3. Dateien in `/etc/opnsense` kopieren und nur auf Root berechtigen:
```bash
sudo mv ... /etc/openvpn
sudo chmod 600 /etc/openvpn/...
```

4. OpenVPN-Konfigurationsdatei anpassen:
```bash
sudo vim /etc/openvpn/config.ovpn

# check config
pkcs12 /etc/openvpn/vpn.p12
tls-auth /etc/openvpn/vpn.key 1
script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
EOF
```

5. Script erstellen:
```bash
sudo vim /etc/openvpn/addproxy.sh
sudo vim /etc/openvpn/remproxy.sh
```

6. Script `/etc/openvpn/update-resolv-conf` anpassen:
```bash
sudo vim /etc/openvpn/update-resolv-conf

## case up
su -l ${pc-username} -c /etc/openvpn/addproxy.sh

## case down
su -l ${pc-username} -c /etc/openvpn/remproxy.sh
```

## Monitoring
### Install Node-Exporter
Downloadsite Node-Exporter: [Link](https://prometheus.io/download/#node_exporter)

1. Create directory and install Node-Exporter:
```bash
mkdir -p ~/workspace/docker_monitoring/
cd ~/workspace/docker_monitoring
wget https://github.com/prometheus/node_exporter/releases/download/v1.0.0/node_exporter-1.0.0.linux-amd64.tar.gz
tar -zxf node_exporter-0.18.1.linux-amd64.tar.gz
mv node_exporter-0.18.1.linux-amd64 node_exporter
```
2. Create systemd-unitfile:
```bash
sudo vim /etc/systemd/system/node_exporter.service
```

3. Enable and start Node-Exporter:
```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```
4. Test Webaccess: http://localhost:9100

### Install Prometheus / Grafana / Cadvisor
1. Download und start docker-compose:
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo systemctl enable docker && sudo systemctl start docker
```
2. Create directory:
```bash
mkdir -p ~/workspace/docker_monitoring/{pr_config,gr_config}
```
3. Create docker-compose-file:
```bash
sudo vim ~/workspace/docker_monitoring/docker-compose.yml
```

4. Create configfile for Prometheus & Grafana:
```bash
sudo vim ~/workspace/docker_monitoring/pr_config/prometheus.yml
sudo vim ~/workspace/docker_monitoring/gr_config/grafana.ini
```

5. Start docker-compose:
```bash
sudo docker-compose -f ~/workspace/docker_monitoring/docker-compose.yml up -d
```
6. Test Webaccess:
- http://localhost:9090
- http://localhost:9100
- http://localhost:3000

7. Add Datasource and Dashboard

## Debug
### disable resolved-system
1. Edit NetworkManager-Configfilie:
```bash
sudo vim /etc/NetworkManager/NetworkManager.conf
```
2. Add `dns=default` in the `[main]`-Section:
- `...`
- `[main]`
- `dns=default`
- `...`
3. Restart NetworkManager:
```bash
sudo systemctl restart NetworkManager
```
4.  Check:
```bash
sudo cat /etc/resolv.conf
dig google.ch
```

---
links: [[100 Linux MOC|Linux MOC]] - [[000 Index|Index]]