tags: #wireshark

# Wireshark

links: [[100 Linux MOC|Linux MOC]] - [[000 Index|Index]]

---

# remote capturing

```bash
# create named pipe
mkfifo /tmp/remote
# start wireshark
wireshark -k -i /tmp/remote
# run tcpdump on server via ssh (e.g. for interface eth0)
ssh ${USER}@${IP} "sudo tcpdump -s 0 -U -n -w - -i eth0 not port 22" > /tmp/remote
```

---
links: [[100 Linux MOC|Linux MOC]] - [[000 Index|Index]]