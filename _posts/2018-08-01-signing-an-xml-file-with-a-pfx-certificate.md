---
title: Signing an XML file with a .pfx certificate
layout: post
comments: true
published: false
description: 
keywords: php, xmldsig, xml, dsig, pfx, crt, certificate
---

Digital signatures are important because they provide end-to-end message integrity guarantees, and can also provide authentication information about the originator of a message. In order to be most effective, the signature must be part of the application data, so that it is generated at the time the message is created, and it can be verified at the time the message is ultimately consumed and processed.

I just implemented a small library [odan/xmldsig](https://github.com/odan/xmldsig) in PHP to sign and validate XML files.

Reade more: [Understanding XML Digital Signature](https://msdn.microsoft.com/en-us/library/ms996502.aspx)

