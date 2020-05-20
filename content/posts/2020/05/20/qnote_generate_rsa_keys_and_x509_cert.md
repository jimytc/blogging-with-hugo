---
title: "Generate RSA key pairs with encryption"
date: 2020-05-20T23:00:49+08:00
tags: [openssl,rsa,certificate]
categories: [engineering]
---

I've spent some time investigating how to create RSA key pairs for a feature.

Put a note here just in case I need it a gain.

### Generate RSA private key with 2048 bits

```sh
$ openssl genrsa -out private.pem 2048
```

### Generate RSA private key with AES-256 encryption

```sh
$ openssl genrsa -aes256 -out private.pem 2048
```

Note that we need to offer a at least 4-bytes passphrase in order to use AES-256 encryption.

### Extract public key from the key pair

```sh
$ openssl rsa -in private.pem -pubout -out public.pem
```

### Generate x509 certificate with generated private key

```sh
$ openssl req -new -x509 -key private.pem -out cert.pem
```

Now we have the private key, public key and a x509 certificate.
