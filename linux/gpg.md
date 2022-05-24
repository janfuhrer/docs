# docs: linux/gpg
## create key
1. Schlüssel erstellen
- (1) RSA und RAS
- Schlüssellänge: 4096
- Gültig: 0 = Schlüssel verfällt nie
- Name: ...
- E-Mail: ...
- Kommentar: GPG-Key for encrypted Backups
```bash
gpg --full-gen-key
```

2. Export Public-Key
```bash
gpg -o backup_pubkey.gpg --armor --export <email>
```

3. Export Private-Key
```bash
gpg -o backup_privkey.gpg --armor --export-secret-key <email>
```

## encrypt/ decrypt files
Encrypt Files
```bash
gpg -o test.txt.gpg -e --recipient <email> test.txt
```

Decrypt Files
```bash
gpg -o test.txt -d test.txt.gpg
```

## Import Public-Key
1. Import
```bash
gpg --import ~/backup_pubkey.gpg
```
2. Trust Key
```bash
gpg --edit-key <email>
> trust
> 5
> save
```