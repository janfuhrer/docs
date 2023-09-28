tags: #linux #security #openssl

# OpenSSL

links: [[100 Linux MOC|Linux MOC]] - [[400 Security MOC|Security MOC]] - [[000 Index|Index]]

---
## DHparam

- pre-compute a *safe prime* ($p=2q+1$) for the DH key exchange for every handshake with a client
- the purpose is to customize the parameters and use the own ones
- finding such prime is really computational intense and has to be pre-computed
- this parameter can be published (is sent out for every key-exchange)

```bash
openssl dhparam -out dhparam.pem 4096
```

## PKCS / X509

```bash
# pkcs12
openssl pkcs12 -nokeys -info -in cert.p12 -passin pass:${password}

# x509
openssl x509 -in cert.crt -text -noout

# x509: get specific fields
openssl x509 -in cert.crt -subject -issuer -startdate -enddate -noout

# der to base64
openssl x509 -inform der -in cert-der.crt -out cert-base64.crt
```

- get tls certificate of remote host
```bash
echo | openssl s_client -showcerts -servername ${host} -connect ${host}:443 2>/dev/null | openssl x509 -inform pem -noout -text
```


---
links: [[100 Linux MOC|Linux MOC]] - [[400 Security MOC|Security MOC]] - [[000 Index|Index]]