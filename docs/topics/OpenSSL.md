tags: #linux #security #openssl

# OpenSSL

links: [[100 Linux MOC|Linux MOC]] - [[400 Security MOC|Security MOC]] - [[000 Index|Index]]

---

## Security/Server Side TLS

- https://wiki.mozilla.org/Security/Server_Side_TLS
- https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html
- https://cipherlist.eu/
- Nginx: https://kubernetes.github.io/ingress-nginx/deploy/hardening-guide/

**Test**

- https://www.ssllabs.com/ssltest/analyze.html
- https://cryptcheck.fr/
- https://testssl.sh

## DHparam

- pre-compute a *safe prime* ($p=2q+1$) for the DH key exchange for every handshake with a client
- the purpose is to customize the parameters and use the own ones
- finding such prime is really computational intense and has to be pre-computed
- this parameter can be published (is sent out for every key-exchange)

> The dhparam file contains the prime which defines the group for the DH key exchange. It is not a secret, and will be sent in clear during the key exchange, so there is no point in trying to keep it secret.

```bash
openssl dhparam -out dhparam.pem 4096
```

## Decode/deserialise DHparam

> [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) files are base64-encoded [DER](https://en.wikipedia.org/wiki/X.690#DER_encoding) serialised [ASN.1](https://en.wikipedia.org/wiki/Abstract_Syntax_Notation_One) files with a header and footer guard. A ASN.1 parser with the right schema can decode them.

```bash
# use openssl
openssl asn1parse <dhparam.pem
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