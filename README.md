SCHANNEL
========

[![Build Status](https://travis-ci.org/deekayen/ansible-role-schannel.svg?branch=master)](https://travis-ci.org/deekayen/ansible-role-schannel)

Makes changes to SCHANNEL settings of the Windows
registry to serve more secure cryptographic
communications for services like IIS and WinRM.

Ciphers:

* AES 128/128
* AES 256/256
* DES 56/56
* NULL
* RC2 40/128
* RC2 56/128
* RC2 128/128
* RC4 40/128
* RC4 56/128
* RC4 64/128
* RC4 128/128
* Triple DES 168/168

Protocols:

* PCT 1.0
* SSLv2
* SSLv3
* TLS 1.0
* TLS 1.1
* TLS 1.2

Hashes:

* MD5
* SHA-1
* SHA-256
* SHA-384
* SHA-512

Related documentation:

* [How to restrict the use of certain cryptographic algorithms and protocols in Schannel.dll](https://support.microsoft.com/en-us/help/245030/how-to-restrict-the-use-of-certain-cryptographic-algorithms-and-protocols-in-schannel.dll)
* [Schannel Security Support Provider Technical Reference: TLS/SSL Settings](https://technet.microsoft.com/en-us/library/dn786418.aspx)
* [MS16-065: Description of the TLS/SSL protocol information disclosure vulnerability (CVE-2016-0149): May 10, 2016](https://support.microsoft.com/en-us/help/3155464/ms16-065-description-of-the-tls-ssl-protocol-information-disclosure-vu)
* [Enabling strong cryptography for all .Net applications](https://www.johnlouros.com/blog/enabling-strong-cryptography-for-all-dot-net-applications)
* [Updated Support for Diffie-Hellman Key Exchange](https://docs.microsoft.com/en-us/security-updates/SecurityAdvisories/2016/3174644)

Requirements
------------

Windows

Role Variables
--------------

Default values are as follows:

```
schannel_dhmodulus: "00000800"

schannel_3des: false
schannel_aes_128: true
schannel_aes_256: true
schannel_des: false
schannel_null: false
schannel_rc2: false
schannel_rc4: false

schannel_md5: false
schannel_sha1: true
schannel_sha256: true
schannel_sha384: true
schannel_sha512: true

schannel_pct: false
schannel_sslv2: false
schannel_sslv3: false
schannel_tlsv10: false
schannel_tlsv11: true
schannel_tlsv12: true

schannel_usestrongcrypto: true
```

Valid values for schannel_dhmodulus:
* 1024 = 00000400
* 2048 = 00000800
* 3072 = 00000C00
* 4096 = 00001000

Example Playbook
----------------

    - name: Harden Windows SCHANNEL configurations.
      hosts: windows2012

      vars:
        schannel_tlsv10: true

      roles:
        - deekayen.schannel

Dependencies
------------

None.

License
-------

BSD 3-Clause License
