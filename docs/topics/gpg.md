tags: #linux #gpg

# gpg

links: [[100 Linux MOC|Linux MOC]] - [[000 Index|Index]]

---
## create key

1. Schlüssel erstellen
```bash
gpg --full-gen-key
```

2. Export Public-Key
```bash
gpg -o backup_pubkey.gpg --armor --export ${email}
```

3. Export Private-Key
```bash
gpg -o backup_privkey.gpg --armor --export-secret-key ${email}
```

## encrypt/ decrypt files

Encrypt Files

```bash
gpg -o test.txt.gpg -e --recipient ${email} test.txt
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
gpg --edit-key ${email}
> trust
> 5
> save
```

## Verify signature

- with keyring

```bash
# import public key
gpg --import pubkey.asc

# export key to keyring
gpg --output ./test.keyring --export 0x${FP_KEY}

# verify a file
gpgv --keyring=./test.keyring ${PACKAGE}.asc ${PACKAGE}
```

- without keyring

```bash
# import public key
gpg --import pubkey.asc

# verify signature
gpg --verify ${FILE}.sign
```

---
links: [[100 Linux MOC|Linux MOC]] - [[000 Index|Index]]