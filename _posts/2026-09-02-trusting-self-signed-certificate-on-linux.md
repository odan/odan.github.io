---
title: Trusting a self-signed localhost certificate in Chrome on Linux
layout: post
comments: true
published: true
description: Trust a self-signed localhost HTTPS certificate in Chrome on Ubuntu, Linux Mint, and other Debian-based Linux distributions.
keywords: chrome, localhost, https, ssl, tls, self-signed certificate, linux, ubuntu, linux mint, certutil, nss
---

When developing locally over HTTPS, Chrome may reject a self-signed certificate with:

```
NET::ERR_CERT_AUTHORITY_INVALID
```

For example, a development profile might serve HTTPS using a certificate stored at:

```text
src/main/resources/certs/localhost.pem
```

On Ubuntu, Linux Mint, and other Debian-based distributions, you can trust the certificate in Chrome by adding it to the user's NSS certificate database.

## Install the NSS tools

Install `certutil`:

```bash
sudo apt install libnss3-tools
```

## Create the NSS certificate database

Chrome uses an NSS database for user certificates. Create it if it does not already exist:

```bash
mkdir -p "$HOME/.pki/nssdb"
certutil -d sql:"$HOME/.pki/nssdb" -N --empty-password
```

If the database already exists, the second command may not be necessary.

## Add the localhost certificate

From the project root, import the development certificate:

```bash
certutil \
  -d sql:"$HOME/.pki/nssdb" \
  -A \
  -t "P,," \
  -n "localhost" \
  -i src/main/resources/certs/localhost.pem
```

You can verify that it was added with:

```bash
certutil -d sql:"$HOME/.pki/nssdb" -L
```

You should see an entry named `localhost`.

## Restart Chrome

Close Chrome completely, including any background processes, and start it again.

Then open:

```text
https://localhost
```

Chrome should now accept the local certificate without showing `NET::ERR_CERT_AUTHORITY_INVALID`.

## Removing the certificate

If you no longer need the certificate, remove it from the NSS database with:

```bash
certutil -d sql:"$HOME/.pki/nssdb" -D -n "localhost"
```

This setup is intended for local development only. Self-signed localhost certificates should not be used as a replacement for certificates issued by a trusted certificate authority in production.
