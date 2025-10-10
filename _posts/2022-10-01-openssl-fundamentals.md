---
layout: post
title: OpenSSL Fundamentals, Encryption Notes
categories: [ Cryptography ]
tags: [RSA, Encryption, OpenSSL]
enable_toc: true
render_with_liquid: true
date: 2022-10-01 00:00:00 +0530
---


# OpenSSL
**OpenSSL** is a software library for applications that secure communications over computer networks. Is a C library that implements the main cryptographic operations like symmetric encryption, public-key encryption, digital signature, hash functions, etc.

**OpenSSL** contains an open-source implementation of the [SSL and TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) protocols.

**The OpenSSL Software Foundation (OSF)** represents the OpenSSL project in most legal capacities including contributor license agreements, managing donations, and so on. OpenSSL Software Services (OSS) also represents the OpenSSL project, for Support Contracts.

**OpenSSL** is available for most Unix-like operating systems (including Linux, macOS, and BSD) and Microsoft Windows.

The source code can be downloaded from [www.openssl.org](https://www.openssl.org)

There are several forks of OpenSSL project.

-   [LibreSSL](https://www.libressl.org/) is a version of the TLS/crypto stack forked from OpenSSL in 2014, with goals of modernizing the codebase, improving security, and applying    best practice development processes.

-   [BoringSSL](https://boringssl.googlesource.com/boringssl/) is a fork of OpenSSL that is designed to meet Google's needs. Although BoringSSL is an open source project, it is not intended for general use, as OpenSSL is.

# Installing OpenSSL

## On Ubuntu Linux distribution

On Ubuntu Linux distribution, OpenSSL is usually already available; however, it can be installed using the following commands:
```sh
$ sudo apt-get install openssl
```

## On Windows

- Install [Chocolatey](https://chocolatey.org/install)
- To install OpenSSL - The Open Source SSL and TLS toolkit, run the following command from the command line or from PowerShell:
```sh
choco install openssl.light --version=1.0.1.3
```

## Checking the version of OpenSSL
```sh
$ openssl version -a
```

# OpenSSL commands

To list OpenSSL commands run the following command on a terminal.
```sh
$ openssl list-standard-commands
```

See a brief description of each command: [STANDARD COMMANDS](https://www.openssl.org/docs/man1.0.2/man1/openssl.html)


# OpenSSL secret key encryption algorithms
To list OpenSSL secret key encryption algorithms run the following command on a terminal.
```sh
$ openssl list-cipher-commands
```
See a brief description of each command: [ENCODING AND CIPHER COMMANDS](https://www.openssl.org/docs/man1.0.2/man1/openssl.html)

# How to encrypt and decrypt using AES

## Encrypting Using Passwords
OpenSSL makes it easy to encrypt/decrypt files using a passphrase. Unfortunately, pass phrases are usually "terrible"
and difficult to manage and distribute securely.
```sh
$ openssl aes-256-cbc -in secret.txt -out secret.txt.enc
...
enter aes-256-cbc encryption password: ****
Verifying - enter aes-256-cbc encryption password: ****
```

**Note** that secret.txt.enc is a binary file; sometimes, it is desirable to encode this binary file into a text format for
compatibility/interoperability reasons. The following command can be used to do that:
```sh
$ openssl enc -base64 -in secret.txt.enc -out secret.txt.enc.b64
$ cat secret.txt.enc.b64
...
U2FsdGVkX1+k+exECYX1O5lFTDRy7f5+2RweZGILQTM=
```

In order to decode from base64, the following commands are used. Take the secret.txt.enc.b64 file from the previous example:

```sh
$ openssl enc -d -base64 -in secret.txt.enc.b64 -out secret.txt.enc

```

## Decrypting Using Passwords
```sh
$ openssl aes-256-cbc -d -in secret.txt.enc -out secret.txt.dec
...
enter aes-256-cbc decryption password: ****
12345
```

## Encrypting Using pseudo-random bytes key.
```sh
$ openssl rand 192 -out key
$ openssl aes-256-cbc -in secret.txt -out secret.txt.enc -pass file:key
$ tar -zcvf secret.tgz *.enc
```

## Decrypting Using pseudo-random bytes key.
```sh
$ tar -xzvf secret.tgz
$ openssl aes-256-cbc -d -in secret.txt.enc -out secret.txt.dec -pass file:key
```

## Sharing the file:key using [rsautl](https://www.openssl.org/docs/man1.0.2/man1/rsautl.html)
Assuming that id_rsa.pub is the public key you want to use and that id_rsa is the private key the recipient will use, and secret.txt
is the data you want to transmit…

### Encrypting
```sh
$ openssl rand 192 -out key
$ openssl aes-256-cbc -in secret.txt -out secret.txt.enc -pass file:key
$ openssl rsautl -encrypt -pubin -inkey ~/.ssh/id_rsa.pub -in key -out key.enc
$ tar -zcvf secret.tgz *.enc
```

### Decrypting
```sh
$ tar -xzvf secret.tgz
$ openssl rsautl -decrypt -ssl -inkey ~/.ssh/id_rsa -in key.enc -out key
$ openssl aes-256-cbc -d -in secret.txt.enc -out secret.txt.dec -pass file:key
```

# How to encrypt and decrypt using RSA

## Generating the private key
```sh
$ openssl genpkey -algorithm RSA -out privatekey.pem -pkeyopt rsa_keygen_bits:1024
............++++++
........++++++
```

After executing the command, a file named privatekey.pem is produced, which contains the generated private key. This is shown as follows:

```sh
$ cat privatekey.pem
-----BEGIN PRIVATE KEY-----
MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBANQEftv/f2oBSXiA
oozCPIzRjnPQgqDF+fZX9A6WI09AI6NVXqAsIEJ1aNxQ8tgHzHXIw2Yika4FxyCd
ds3qpz2WFOEZK2YKrR3Vd/KzA2CyvMl4DXngJhFlSKbPUq8BHrz0YLfC1uhFtWeK
4IFBFM55vIyaRDroYg7pOCxw12n5AgMBAAECgYAH9H86C/Ug6hlynwj6VRNmiTpa
GBm+pI8DcjtjDLkYcSSlLT/WrLEtLTCZC6SA/JHsXXMPcv6aU/crvxzFDyflMy9O
8/Z/04Ll7tu3qXm5gvmqDS9CDiLyWZnrlnjeYzkq1l0nvt+G6Ngs+/uoum+A+y7g
d0uaF55hmKuF5nusvQJBAOxYs3VvQkAq1rhzUGSnC8rpZee0UXQlEyKeU7KSFG5y
HLcdH8OrBxzPpTgMP9H33c4FYKvLUdWEXUM4e2VnVn8CQQDlpeKMyMI9yND6Ewz+
/Zhl/EziWkcyVAUttixX8SHnJtd3C11G+iOKpTr3qw8j7GH437S171giHte/E4RH
IrOHAkEAzvG78Q/CSr031bnior9BrCJBgGh7Cd+MqbtIPgt6qFpymkN+FK4kRC3s
1O6k0wzdwg8jXklhFjwYDUvfgCLDsQJBAOM6BjYjFv8nSo+Gdh+AMWEICdMWXMgR
lqYqUSoa797V8fBakEsAilZPM0+INIzpAe/M+fPjBSONvQ/VcdcpINUCQBuSUK41
CV5dQ0tQ48qU99PU5yTRAgn8TTVndZVE7du3d+hiSmn3XB3LzkrFKJKwojRqt4JV
hctcECz3797r3fQ=
-----END PRIVATE KEY-----
```

## Generating the public key

As the private key is mathematically linked to the public key, it is possible to generate or derive the public key out of
the private key. Taking the example of the preceding private key, the public key can be generated as shown here:

```sh
$ openssl rsa -pubout -in privatekey.pem -out publickey.pem
writing RSA key
```

Public key can be viewed using a file reader or any text viewer, as shown here:
```sh
$ cat publickey.pem
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDUBH7b/39qAUl4gKKMwjyM0Y5z
0IKgxfn2V/QOliNPQCOjVV6gLCBCdWjcUPLYB8x1yMNmIpGuBccgnXbN6qc9lhTh
GStmCq0d1XfyswNgsrzJeA154CYRZUimz1KvAR689GC3wtboRbVniuCBQRTOebyM
mkQ66GIO6TgscNdp+QIDAQAB
-----END PUBLIC KEY-----
```

## Encrypting
Taking the public key generated, the command to encrypt a text file secret.txt can be constructed, as shown here:
```bash
openssl rsautl -encrypt -inkey publickey.pem -pubin -in secret.txt -out secret.txt.rsa.enc
```

## Decrypting
In order to decrypt the RSA-encrypted file, using the private key, the following command can be used:
```bash
openssl rsautl -decrypt -inkey privatekey.pem -in secret.txt.rsa.enc -out secret.txt.rsa.dec
```

##### Sources

- <https://www.scottbrady.io/openssl/creating-rsa-keys-using-openssl>
- <https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs>

