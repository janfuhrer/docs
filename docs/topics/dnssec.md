ags: #dns #dnssec

# DNSSEC

links: [[100 Linux MOC|Linux MOC]] - [[000 Index|Index]]

---

See following articles:
- https://www.cloudflare.com/dns/dnssec/how-dnssec-works/
- https://www.netmeister.org/blog/doh-dot-dnssec.html

## Overview

- security extensions for verifying the identity of DNS root servers and authoritative nameservers
- prevent DNS cache poisoning (or DNS spoofing)
- uses public key cryptography to verify and authenticate data
- **no encryption of communication**
- DNSSEC only addresses the question of data integrity and authenticity, but does not in any way concern itself with the aspects of confidentiality.

## Key hierarchy

- hierarchical digital signing policy across all layers of DNS (a root DNS server would sign a key for `.com` nameserver, `.com` nameserver would then sign a key for `google.com`)
- To close the chain of trust, the root zone itself needs to be validated (proven to be free of tampering or fraud), and this is actually done using human intervention.

## Record types

DNS record types
- **RRSIG**: contains a cryptographic signature
- **DNSKEY**: contains a public signing key
- **DS**: contains the hash of a DNSKEY record
- **NSEC** and NSEC3: for explicit denial-of-existence of a DNS record
- **CDNSKEY** and CDS: for a child zone requesting updates to DS record(s) in the parent zone

## Procedure

1. group all the records with the same type into a resource record set (**RRset**)
2. the full RRset gets digitally signed with the *zone-signing-key pair* (**ZSK**) -> this means that you must request and validate all of the "type of records" from a zone instead of validating only one of them
3. The RRset is signed using the private ZSK and stores them in a **RRSIG** record
4. The public ZSK is published in a **DNSKEY** record
5. a resolver can use the RRset, RRSIG and public ZSK to validate a response

## Key-Signing-Key

- a key-signing key (KSK) validates the DNSKEY record -> it signs the public ZSK (which is stored in a DNSKEY record), creating an RRSIG for the DNSKEY
- it is difficult to swap out an old or compromised KSK, changing the ZSK is much easier on the other hand

## Delegation Signer Records

- a delegation signer (DS) record allow the transfer of trust from a parent zone to a child zone.
- a zone operator hashes the DNSKEY record containing the public KSK and gives it to the parent zone to publish a DS record -> a change in the KSK also requires a change in the parent zone's DS record

## Explicit Denial of Existence

- If you ask DNS for the IP address of a domain that doesn't exist, it returns an empty answer -> this is a problem if you want to authenticate the response.
- NSEC and NSEC3 record types works by returning the "next secure" record (e.g. if alphabetically sorted, respond with the next record in the zone) -> this solution allows anybody to walk through the zone and gather every single record without knowing which ones they're looking for. 

---
links: [[100 Linux MOC|Linux MOC]] - [[000 Index|Index]]