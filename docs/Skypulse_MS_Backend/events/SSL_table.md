---
sidebar_position: 4
---

# SSL Logs Table Field 

### Field-by-Field Explanation

#### 1. `hostname`

* The hostname the certificate was issued for.
* Example: `api.example.com`



#### 2. `issuer`

* Certificate Authority (CA) that issued the certificate.
* Example: `Let's Encrypt Authority X3`



#### 3. `serial_number`

* Globally unique ID assigned per certificate by the issuer.
* Example: `04:AF:39:1C:...`



#### 4. `signature_algorithm`

* The algorithm used to sign the certificate.
* Examples: `SHA256withRSA`, `ECDSAwithSHA384`



#### 5. `public_key_type`

* Type of public key.
* Examples: `RSA`, `EC`, `DSA`



#### 6. `public_key_size`

* Size of the public key.
* Examples:

```
2048 (RSA)
4096 (RSA)
256 (EC P-256)
```



#### 7. `subject_alternative_names`

* Comma-separated list of domains the certificate covers.
* Example: `www.example.com, api.example.com, example.com`



#### 8. `is_chain_valid`

* Boolean indicating if the certificate chain is complete and trusted.
* Values: `true` or `false`



#### 9. `subject`

* The certificate’s self-claimed identity.
* Example: `CN=api.example.com, O=Example Corp, C=US`



#### 10. `fingerprint`

* Unique hash of the entire certificate.
* Examples:

```
SHA-1 fingerprint
SHA-256 fingerprint
```
