# tutorial-openssl

OpenSSL is a tool used to create and manage certificates. We use it usually to manage SSL/TLS certificates. I'll list here the common commands a devops person usually uses.

Generate a private key:

```sh
openssl genrsa -out my-key.key 2048
```

From the private key we can extract the public key:

```sh
openssl rsa -in my-key.key -pubout -out my-key.pub
```

In asymmetric cryptography the private key is used for decryption and signing while the public key is used for encryption and signature validation. As the name and the functions suggest, the private key must be kept secret, while the public key can be distributed. The public key can be generated from the private key, but the private key cannot be generated from the public key.

In creating a asymmetric communication with SSL we need to verify that the keys used are indeed from the source we are trying to connect, for example, if we are trying to connect to *google.com* we need to be sure the keys it sends to us are indeed from *google.com*. We achieve this with a chain of trust, where we have an authority, called **Certification Authority**, that has private key, which signs a document called certificate. Among the data in the certificate we have the *common name (CN)*, the *public key* and the *signature*. With the CA public key we can verify if the certificate was indeed signed by it, and as we trust the CA, we trust the signature and we trust the key in the certificate is indeed from the *common name (CN)*, which could be *google.com* for example.

Create a CA private key:
```sh
openssl genrsa -out ca.key 2048
```

Create a CA certificate:
```sh
# Here x509 is a certificate format
openssl req -x509 -new -nodes -key ca.key -subj "/CN=cadomain.com/O=cadomain" -days 3650 -out ca.crt
```

View the data in the CA certificate:
```sh
openssl x509 -in ca.crt -text -noout
```

Opening the certificate we see it has a signature that was made with the CA private key. Distributing the CA public key, we can validate it was indeed signed by the private key.

```yaml
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            36:58:55:5f:20:33:b1:8f:25:f0:40:3e:0b:dd:3c:b4:73:7e:e4:36
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = cadomain.com, O = cadomain
        Validity
            Not Before: May 10 10:57:11 2023 GMT
            Not After : May  7 10:57:11 2033 GMT
        Subject: CN = cadomain.com, O = cadomain
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:bb:02:a1:fa:e9:d2:6a:6a:42:0c:69:36:e9:21:
                    a6:4c:95:81:db:f4:31:a6:4e:95:66:86:21:4e:b0:
                    e2:59:cf:c7:45:29:ab:d2:90:18:78:e3:a8:be:d8:
                    74:d3:52:3d:3b:0d:5c:fd:92:58:79:bf:b1:46:94:
                    34:f3:c2:0f:10:7f:2f:51:3d:68:a6:6e:b9:7e:a2:
                    21:0e:9b:75:d5:2a:dd:9d:77:41:80:4c:15:c3:d9:
                    4b:ea:6e:71:a9:ba:43:59:14:93:41:64:0d:7d:3b:
                    a3:2f:9b:04:ec:1d:08:53:a4:80:ca:3c:2d:c1:7e:
                    7c:7c:fd:87:0a:00:ff:d5:c0:3f:e9:7f:51:7c:54:
                    3d:98:38:47:a9:fd:17:b3:3d:7b:da:90:59:66:a3:
                    4c:05:42:db:a0:a3:0d:64:2d:c4:a6:35:f3:45:13:
                    9f:1b:26:45:52:5b:71:a8:a1:32:9e:58:0b:56:a2:
                    16:54:a2:42:86:94:ce:71:84:ac:32:80:af:c9:2e:
                    65:a9:24:be:fb:10:29:7b:d4:01:75:ca:54:c1:e4:
                    3f:14:81:05:35:84:64:be:49:97:78:cb:7a:67:e8:
                    ab:2c:12:16:14:41:99:be:99:3d:d7:90:b7:6e:2f:
                    aa:44:e5:c2:58:b6:12:9e:39:cf:d5:fd:10:b9:50:
                    47:d7
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                88:54:BB:95:89:68:E4:B4:65:15:D1:C6:37:4B:11:D5:E4:48:5F:AA
            X509v3 Authority Key Identifier: 
                88:54:BB:95:89:68:E4:B4:65:15:D1:C6:37:4B:11:D5:E4:48:5F:AA
            X509v3 Basic Constraints: critical
                CA:TRUE
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        3f:0f:12:13:d1:67:9f:9c:e7:ae:73:36:ee:bc:7f:1f:0e:3c:
        bb:a7:ed:90:9d:ea:b0:c0:4c:d4:31:e6:21:a0:6f:22:31:45:
        c5:b1:3e:b0:cb:82:3f:65:71:30:d9:7f:91:5b:13:cd:bb:5a:
        85:0f:a2:e3:c7:f7:28:2c:a3:8a:15:0e:bf:6d:b6:c9:18:47:
        34:3f:d8:66:8a:64:f0:b2:d4:e7:78:6e:64:33:d0:86:f1:a1:
        53:bc:3c:db:5a:ab:f7:dc:72:87:05:9a:15:fe:1c:76:08:26:
        61:a2:92:af:e3:6d:41:34:66:e3:44:dc:b9:dd:9e:4a:78:4f:
        35:d1:a9:14:b2:6d:ff:7e:0c:00:3a:11:f1:e0:67:96:37:38:
        cb:fd:1c:59:45:a4:44:1c:3b:f1:ef:da:d7:09:ab:69:43:b3:
        61:f7:9a:9f:2f:4b:4c:9f:5f:c1:42:b6:e5:2a:41:aa:cf:71:
        34:2b:65:b7:52:51:d3:70:51:f2:d7:83:74:14:f7:18:54:8e:
        fa:f0:76:be:ce:81:6e:a8:0f:dd:34:83:e3:c6:63:94:92:69:
        9b:38:31:78:9f:c5:3a:6e:ee:7e:8e:30:a3:da:a7:3f:70:4f:
        75:ed:b3:33:53:8c:be:25:78:f0:7d:78:5d:d9:ed:01:79:ed:
        2f:c9:22:62
```

To create a certificate based on keys and signed by a CA we create a *Certificate Signing Request (CSR)*, which is a formal request to issued a certificate for a particular public key.

Create CSR:
```sh
openssl req -new -key my-key.key -out my-csr.csr -subj "/CN=mydomain.com/O=mydomain"
```

Sign the CSR by the CA:

```sh
openssl x509 -req -in my-csr.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out my-cert.crt -days 365
```