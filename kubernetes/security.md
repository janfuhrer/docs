# docs: kubernetes/security
#kubernetes #security

## nuclei
https://github.com/projectdiscovery/nuclei/

- security check for website
- scanning for a variety of protocols

```bash
nuclei -u example.com
```

## SSLLabs
https://www.ssllabs.com/ssltest

- check TLS configuration of website

## FairWindsOps
#nova #pluto #goldilocks #polaris

## nova
https://github.com/FairwindsOps/nova

```bash
nova find --wide
```

## pluto
https://github.com/FairwindsOps/pluto

```bash
pluto detect-helm -owide
```

## goldilocks
https://github.com/FairwindsOps/goldilocks


## polaris
https://github.com/FairwindsOps/polaris

```bash
polaris dashboard --port 8080
```