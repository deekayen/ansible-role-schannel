TLS
===

Makes changes to the Windows registry to serve more secure cryptographic
communications for services like IIS and winrm.

Enable ciphers:

* AES 128/128
* AES 256/256
* Triple DES 168/168

Enable protocols:

* TLS 1.1
* TLS 1.2

Disable hashes:

* MD5

Disable ciphers:

* DES 56/56
* NULL
* RC2 40/128
* RC2 56/128
* RC2 128/128
* RC4 40/128
* RC4 56/128
* RC4 64/128
* RC4 128/128

Disable protocols:

* PCT 1.0
* SSLv2
* SSLv3
* TLS 1.0

Requirements
------------

Windows

Role Variables
--------------

Nothing yet.

Example
-------

    - name: Harden SSL and TLS configurations.
      hosts: windows2008

      roles:
        - tls

Dependencies
------------

None.

License
-------

Unlicense.
