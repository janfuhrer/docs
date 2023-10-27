ags: #dns 

# Name System

links: [[100 Linux MOC|Linux MOC]] - [[000 Index|Index]]

---

## Security Goals for Name Systems

- Query origin anonymity (some anonymity with recursive resolver)
- Data origin authentication and integrity protection (DNSSEC)
- Zone confidentiality (DNS: yes, DNSSEC: no)
- Query and response privacy (DoT, DoH, DNSCrypt)
- Censorship resistance (Namecoin)
- Traffic amplification resistance
- Availability

## DNS suffering

- DNS was not designed with security in mind and contains several design limitations
- These limitations, combined with advances in technology, make DNS servers vulnerable to a broad spectrum of attacks.
- Since DNS is an integral part of most Internet requests, it can be a prime target for attacks
- This lack of privacy has an impact on security and, in some cases, human rights; if DNS queries are not private, then it becomes easier for governments to censor the Internet and for attackers to stalk users' online behaviour.

- DNS remains a source of traffic amplification DDoS
- DNS censorship causes collateral damage in other countries (i.e. by China)
- DNS is part of the mass surveillance apparatus
- DNS is abused for offensive cyber war
- DoT, DoH, DNSSEC, ... will not fix this

## DNS attacks

- **DNS spoofing/cache poisoning**
	- entering false information into a DNS cache which results in incorrect response and false direction of user to websites
	- UDP is vulnerable to forging responses (no handshake as in TCP) -> no way to verify the response
	- Instead of going to the correct website, traffic can be diverted to a malicious machine or anywhere else the attacker desires; often this will be a replica of the original site used for malicious purposes such as distributing malware or collecting login information.
- **DNS tunneling**
	- this attack uses other protocols to tunnel through DNS queries and responses. Attackers can use SSH, TCP or HTTP to pass malware or stolen information into DNS queries, undetected by most firewalls
- **DNS hijacking**
	- In DNS hijacking, the attacker redirects queries to a different domain name server. This can be done either with malware or with the unauthorised modification of a DNS server. Although the result is similar to that of DNS spoofing, this is a fundamentally different attack because it targets the DNS record of the website on the nameserver, rather than a resolver’s cache.
- **NXDOMAIN attack**
	- DNS flood attack with records that do not exist in attempt to cause a denial-of-service for legitimate traffic
	- attack a recursive resolver with the goal of filling the resolver's cache with junk requests
- **Phantom domain attack**
	- similar result to an NXDOMAIN attack
	- Attacker sets up a bunch of 'phantom' domain server that either respond to requests very slowly or not at all. The resolver is then hit with a flood of requests to these domains and the resolver gets tied up waiting for responses, leading to slow performance and denial-of-service.
- **Random subdomain attack**
	- random, nonexistend subdomains of one legitimate site. the goal is to create a denial-of-service for the domain's authoritative nameserver
- **Domain lock-up attack**
	- attackers setting up special domains and resolvers to create TCP connections with other legitimate resolvers. Responses to requests from the target resolver sent back slowly, tying up the resolver's resources.
- **Botnet-based CPE attack**
	- using CPE devices (Customer Premise Equipment, hardware from ISP's). Attackers compromise the CPEs and use it in a botnet to perform random subdomain attacks
- **DNS Amplification**
    - DNS amplification is a Distributed Denial of Service (DDoS) attack in which the attacker exploits vulnerabilities in domain name system (DNS) servers to turn initially small queries into much larger payloads, which are used to bring down the victim’s servers. It is a type of reflection attack. During a DNS amplification attack, the perpetrator sends out a DNS query with a forged IP address (the victim’s) to an open DNS resolver, prompting it to reply back to that address with a DNS response. With numerous fake queries being sent out, and with several DNS resolvers replying back simultaneously, the victim’s network can easily be overwhelmed by the sheer number of DNS responses. Reflection attacks are even more dangerous when amplified. “Amplification” refers to eliciting a server response that is disproportionate to the original packet request sent. To amplify a DNS attack, each DNS request can be sent using the EDNS0 DNS protocol extension, which allows for large DNS messages, or using the cryptographic feature of the DNS security extension (DNSSEC) to increase message size. Spoofed queries of the type “ANY,” which returns all known information about a DNS zone in a single request, can also be used.

## Technologies for more security

### DNSSEC

see [[dnssec]]

### DNS-over-TLS (DoT)

- adds TLS encryption on top of UDP
- TCP port 853 -> metadata for DoT connections
- trust of Certificate of DNS-Server (i.e. CA)
- DoT is not necessarily used between all resolvers and authoritative servers, so the protections provided only apply explicitly to the traffic between the client and the recursive resolver
- authenticity provided by DoT only refers to the server, not the records!

### DNS-over-HTTPS (DoH)

- alternative to DoT
- sent via HTTP or HTTP/2 protocols instead of UDP
- port 443 -> hidden in HTTPs traffic, cannot easily be blocked by administrators

### DoT and DoH considerations

- prevent local governments from manipulating DNS traffic and improve the user's privacy with respect to their ISPs and governments -> However, Google or Cloudflare will see the DNS queries and replies of the users

### DNSCurve

- uses curve25519
- public keys are placed in NS records
- encryption and authentication
- not widely used

### DNSCrypt

- encryption and authentication
- DNSCrypt wraps unmodified DNS traffic between a client and a DNS resolver in a cryptographic construction in order to detect forgery.
- Though it doesn't provide end-to-end security, it protects the local network against man-in-the-middle attacks
- helps to prevent DNS amplification attacks by requiring a question to be at least as large as the corresponding response

### RAINS

- new name system developed by ETH Zurich
- does not change the privacy of DNS and allows the local authorities to block Web sites to improve public safety and enforce local laws

### Namecoin

- name system on the blockchain
- queries are performed against a local copy of the blockchain and thus also private
- no law considerations
- much bandwidth and energy for registration and name resolution

### CAA

- CAA records tell the client which CA is allowed for a domain
- relies on DNS(SEC) security!
- the configured CA must also still be trustworthy

### DANE

- DNS-based Authentication of Named Entities (DANE)
- relies on DNSSEC
- if we already have a secure DNS through DNSSEC, we can add certificate information to the DNS

---
links: [[100 Linux MOC|Linux MOC]] - [[000 Index|Index]]