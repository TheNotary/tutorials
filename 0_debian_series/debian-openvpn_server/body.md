# OpenVPN Server

###### Outline
1. Why OpenVPN is so useful
1. Setting up a server
1. Setting up a client



## Why OpenVPN is so Useful
Have you ever been away from your home, and wished you could download a file from your basement computer, but you never bothered to manually make that computer accessible via port forwarding with your router?  Well, if you have a VPN server sitting somewhere on your home network (and have port forwarded it correctly) you'll be able to connect to the VPN and from their, all of your TCP/IP connections with first got to your VPN server, and then allow you to make connection from there... connections from within your home LAN network!  Sweet right?  Your connection is fully encrypted between your client machine and your VPN server too, so even if you wind up using some old-ass `http` only web servers from within a coffee shop, you still won't be exposed to risk of people snooping on you.


## Setting up a Server

###### Install
```
sudo apt-get install openvpn easy-rsa
sudo cp -r /usr/share/easy-rsa/ /etc/openvpn/easy-rsa
sudo chown -r ${USER} /etc/openvpn/easy-rsa
cd /etc/openvpn/easy-rsa
```

###### Server Certificate Generation
Because your clients need to be able to verify they're connecting to the right server, your server needs a cert.
```
./pkitool --initca
./pkitool --server myserver
openssl dhparam -out dh2048.pem 2048
sudo openvpn --genkey --secret ta.key
```
Great, now you have `ca.crt` and `ca.key` for your server.  

You also have `myserver.crt` and `myserver.key` which you need...  idk why, or if this is true, but these keys are similar to client keys, so I guess your server needs them to like test itself maybe?

Additionally, you have a `dh2048.pem` file.  Idk... you need lots of files with this app... And they like showing them all to you I guess...

Finally you have a `ta.key` which is an add-security, shared secret key that you keep on your server and clients to allow connections to be created.

Ok, once we've got all that, we can start looking at the server.conf file.  This tells our openvpn server how to act, what clients to let in, etc, etc.


(/etc/openvpn/server.conf)
```
#Must haves, search through the file and modify these as needed:
ca ca.crt
cert myserver.crt
key myserver.key

# Add this


```







###### Client Cert and Key generation
Because your server needs to know your client is authorized and allowed on the private network, your server needs to generate some client certs and sign them.  
```
export vars   # You can customize the variables if you'd like to change the key a little
./build-key client1
```

Great, now you have `client1.crt` and `client1.key` for your client to connect with.  Oh, the `.csr` file?  That's an artifact of the certificate generation process, you can just ignore it, but it was used to send a request for generating a crt.  The `.key` file is the thing that lets your client onto the VPN, and the `.crt` is what you present to the vpn server to prove that you're the right machine to allow access.  

Finally, we're going to use these .key and .crt files to write a .ovpn format file for easy connecting:

(client1.ovpn)
```

cat >client1.ovpn <<EOF
client
nobind
dev tun
remote-cert-tls server

remote VPN.SERVERNAME.COM 1194 udp

<key>
$(cat client1.key)
</key>

<cert>
$(cat client1.crt)
</cert>

<ca>
$(cat ca.crt)
</ca>

<tls-auth>
$(cat ta.key)
</tls-auth>

key-direction 1

redirect-gateway def1

EOF
```


```
client
nobind
dev tun
remote-cert-tls server

remote VPN.SERVERNAME.COM 1194 udp

<key>
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQDLP1DCH9csJseH
Rts1OMfN5RxxyFWSyjihQ33KMlRl7rWEu/BWZUgL55ol/IvX9l6zEWJMIoQWQlhE
+a9Z0YXUF52mVEAtIdhbVsKgCbn3xrdiP2rpWWniKS96HJVgMe/Mn2ewW3ArVLJM
V7EU/m2+gmyWKXN6HW20ql8YMErWuTLBDdOjmkrkr3ELdKvgVrJ7BB4vuUWiMAg1
sOJ20LMcUYIZDkmrWRccYJn8r+xEVci3TxzbPsHisypfB3pLCCLqEhLdeNOCLykE
jDlsEsVP4T22tB0jshmy2sVnCWTpJUnC0cs60xfeon7ZUnAG1mTK5D7TDH4cFMDd
6E0VMypjAgMBAAECggEBALKUdDHM5jOd8yyyHkMaG7yV9TMoUcADPFS9R1YUeMGD
RyxUMWzH2tDS80czKfBcQYLW4GaC4Unpi0M3m1Gw3gSnp1YQqr69ASvoBGO3iBXk
HRcPH7HeZUFY7KU/XiRCXC+PU/zJqrn31h1r42TN1MFSOXSLATKnjs/x7lIDhI46
Yn8q5Kn0QP0h3+1ZCA/I3aoCUj+fIPa8JuZ3Di9Q7uBWs9OfxqPH40KnpB43XnEH
MgHUIQZ7l+14IlY38ebA36u60yI0ioJbJLl7OdTu9B6w5WCorpokSsNVQ4Y0VvHm
e4Rm1u9IyVXJRdA7Nscrmulyb/UEfgq0OQLakIiVP+ECgYEA+l5h433+p0fVtU3T
yezyaM7Xf83OPQadnhHlt2o+um5Z5PZFhchy8Ok+lJnN007JfR0LqAn02Ccd77lj
xH5TdDoo/BN0ulGoWWSWemHzuhYvFq85jfHzFloxTv250ZtOSpIz4a9kvkwqdwwA
DSuq6hIk/EhA9LvQFP9XDNn4ZZECgYEAz9Gb+6YU8RwCN1YS2QdoKHV4JYCsEx0W
8tyjtYzw3qKWha7c5z2u9+LJATHN6AlLEkvzbcymJ/yEcn0VhDw4PmcGShNC5Dtu
/8vZQsnwV0fPpTKDbpJ370sOzeAGx6khrL+RjTSrPZw4XDGPZ3SuG9KDXo2IGmZz
fIfYjNuWxrMCgYBjYlnbMyWGA7bqjGVYz4z+W7Uhj3GhueGRYKteXndeC/X1NGku
jP5LcVsdI9yXV7wVxRTedG1T7Fsu0NmwozC/f2LLhXGdkFKSgaJWHFHieXHhwFbJ
aNTE97KBF6jOcqbmZRjhKn2EKLnmncXbdI0Y83DpEElwnKkh3KYSfOfkcQKBgFCW
kgxl/Rz6pYlb4XczvhpiYzL30MKgtzN6iClw/D75gbFZe+RYMS+DTDsgWx6t6+Su
ezmK8Kv06k+TXfKnf5ADV5cGHRxwR7z+CcQylvbhrA39pqYMOmIbEySWyUpHtf1N
VF4TnIwJtnlZ5qhRwOqdGcBi1fKW5BXYgAsvZCqtAoGARVxCxnjoHWDDYvG65/VU
c7+Ri+6yXkxps41ZYtK3M70ibGJ6rD12k/6Jlo6ZRmBt2wZscGcWIdrCyeWscy+F
voLukEs2x+1bfvjzmosYFJ6eEuIWZNGJre0CQTF0qdlrKxEOE2dW6IP3T30nzJCj
aMxyV6Lv1HW9zBTudv92DgQ=
-----END PRIVATE KEY-----
</key>

<cert>
-----BEGIN CERTIFICATE-----
MIIFVjCCBD6gAwIBAgIBATANBgkqhkiG9w0BAQsFADCBrzELMAkGA1UEBhMCVVMx
CzAJBgNVBAgTAkNBMRUwEwYDVQQHEwxTYW5GcmFuY2lzY28xFTATBgNVBAoTDEZv
cnQtRnVuc3RvbjEdMBsGA1UECxMUTXlPcmdhbml6YXRpb25hbFVuaXQxETAPBgNV
BAMTCGNoYW5nZW1lMRAwDgYDVQQpEwdFYXN5UlNBMSEwHwYJKoZIhvcNAQkBFhJt
ZUBteWhvc3QubXlkb21haW4wHhcNMTcwNTMxMTQ0MzAxWhcNMjcwNTI5MTQ0MzAx
WjCBrjELMAkGA1UEBhMCVVMxCzAJBgNVBAgTAkNBMRUwEwYDVQQHEwxTYW5GcmFu
Y2lzY28xFTATBgNVBAoTDEZvcnQtRnVuc3RvbjEdMBsGA1UECxMUTXlPcmdhbml6
YXRpb25hbFVuaXQxEDAOBgNVBAMTB2NsaWVudDExEDAOBgNVBCkTB0Vhc3lSU0Ex
ITAfBgkqhkiG9w0BCQEWEm1lQG15aG9zdC5teWRvbWFpbjCCASIwDQYJKoZIhvcN
AQEBBQADggEPADCCAQoCggEBAMs/UMIf1ywmx4dG2zU4x83lHHHIVZLKOKFDfcoy
VGXutYS78FZlSAvnmiX8i9f2XrMRYkwihBZCWET5r1nRhdQXnaZUQC0h2FtWwqAJ
uffGt2I/aulZaeIpL3oclWAx78yfZ7BbcCtUskxXsRT+bb6CbJYpc3odbbSqXxgw
Sta5MsEN06OaSuSvcQt0q+BWsnsEHi+5RaIwCDWw4nbQsxxRghkOSatZFxxgmfyv
7ERVyLdPHNs+weKzKl8HeksIIuoSEt1404IvKQSMOWwSxU/hPba0HSOyGbLaxWcJ
ZOklScLRyzrTF96iftlScAbWZMrkPtMMfhwUwN3oTRUzKmMCAwEAAaOCAXowggF2
MAkGA1UdEwQCMAAwLQYJYIZIAYb4QgENBCAWHkVhc3ktUlNBIEdlbmVyYXRlZCBD
ZXJ0aWZpY2F0ZTAdBgNVHQ4EFgQU/ysO0chec15vkIScXRX8VbsEM0EwgeQGA1Ud
IwSB3DCB2YAUMQDR1rC/ejuRzrjLafiKa3mnk1+hgbWkgbIwga8xCzAJBgNVBAYT
AlVTMQswCQYDVQQIEwJDQTEVMBMGA1UEBxMMU2FuRnJhbmNpc2NvMRUwEwYDVQQK
EwxGb3J0LUZ1bnN0b24xHTAbBgNVBAsTFE15T3JnYW5pemF0aW9uYWxVbml0MREw
DwYDVQQDEwhjaGFuZ2VtZTEQMA4GA1UEKRMHRWFzeVJTQTEhMB8GCSqGSIb3DQEJ
ARYSbWVAbXlob3N0Lm15ZG9tYWluggkA9UnseykJI2UwEwYDVR0lBAwwCgYIKwYB
BQUHAwIwCwYDVR0PBAQDAgeAMBIGA1UdEQQLMAmCB2NsaWVudDEwDQYJKoZIhvcN
AQELBQADggEBAAxJlPBpLr81e2jB48RmpLigr7tbY3YY7ysc9kCpO+kEPfDkAF++
UpOTH3cdqVtqVbVLMSNykzbn+us3M/nlFh2dH6vdH0AtvE4LdOmj5GEvbW5llviH
H+owF6PcQu/lcg1OHrL5/eVR4g4hiM33BxqmtY72bjxW6CH6DcnuRcSusDqYLZu8
1bzhBqHWWYVa8D9rIYny+ngmzD9MUBC66pz779PR0DBrW+kff3j/b6WgZwb7G9qI
PNTmlsIoqhouxq5v4RDBAUdZ+M8UddFI1dujn1XwUM2RaFWFdr8FwR5ZM8qZuW74
qwZSmaaRUPAR1K93XBox1c55J0WQHEh2+44=
-----END CERTIFICATE-----
</cert>

<ca>
-----BEGIN CERTIFICATE-----
MIIE/TCCA+WgAwIBAgIJAPVJ7HspCSNlMA0GCSqGSIb3DQEBCwUAMIGvMQswCQYD
VQQGEwJVUzELMAkGA1UECBMCQ0ExFTATBgNVBAcTDFNhbkZyYW5jaXNjbzEVMBMG
A1UEChMMRm9ydC1GdW5zdG9uMR0wGwYDVQQLExRNeU9yZ2FuaXphdGlvbmFsVW5p
dDERMA8GA1UEAxMIY2hhbmdlbWUxEDAOBgNVBCkTB0Vhc3lSU0ExITAfBgkqhkiG
9w0BCQEWEm1lQG15aG9zdC5teWRvbWFpbjAeFw0xNzA1MzExNDQyMzJaFw0yNzA1
MjkxNDQyMzJaMIGvMQswCQYDVQQGEwJVUzELMAkGA1UECBMCQ0ExFTATBgNVBAcT
DFNhbkZyYW5jaXNjbzEVMBMGA1UEChMMRm9ydC1GdW5zdG9uMR0wGwYDVQQLExRN
eU9yZ2FuaXphdGlvbmFsVW5pdDERMA8GA1UEAxMIY2hhbmdlbWUxEDAOBgNVBCkT
B0Vhc3lSU0ExITAfBgkqhkiG9w0BCQEWEm1lQG15aG9zdC5teWRvbWFpbjCCASIw
DQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMDF0NpBTJfAkUQ1+x8Byj1RT2f6
zeI++ln3/aLla4TRwzjrZg07V/wszmR9mlW/PhVHx1cP7DYHBo7lUkq94yBrhl0f
0KZcOtLOFOmQ2/ZChFyFyAWKOYLzxpE1MSGBhclZQ46TluXMdGvP0m+4mMrWNn1D
mqqW6THk+9tL9DPgOdwXJZeqHtzOOQpjKaFe302yhfl2Ppggq5XlZGeRZftWHUrf
IuC5+eyN1QgImfV0SZqrRuzEwg/G+lPxVv9NNueHMmI5fGrtokJhBiMzxwjzHP7i
sJuAbDXEsHjS3U66WvKzw+Gjcy6JC+p5JN2gnsLjOHgelbkFYm+8XP49JOECAwEA
AaOCARgwggEUMB0GA1UdDgQWBBQxANHWsL96O5HOuMtp+IpreaeTXzCB5AYDVR0j
BIHcMIHZgBQxANHWsL96O5HOuMtp+IpreaeTX6GBtaSBsjCBrzELMAkGA1UEBhMC
VVMxCzAJBgNVBAgTAkNBMRUwEwYDVQQHEwxTYW5GcmFuY2lzY28xFTATBgNVBAoT
DEZvcnQtRnVuc3RvbjEdMBsGA1UECxMUTXlPcmdhbml6YXRpb25hbFVuaXQxETAP
BgNVBAMTCGNoYW5nZW1lMRAwDgYDVQQpEwdFYXN5UlNBMSEwHwYJKoZIhvcNAQkB
FhJtZUBteWhvc3QubXlkb21haW6CCQD1Sex7KQkjZTAMBgNVHRMEBTADAQH/MA0G
CSqGSIb3DQEBCwUAA4IBAQAmL9nAAmewIqXJJuHQTx+QPtybYv8fVLZQxeaV4F0l
a88Gj30RT0g5iIU/8cJxISD2ncF/IuXsReM7YSAa7cgQDz3EZ7zY3LgUg5gheGDb
zCpRjPTUSGw9cRwDsUvdAc+Y0ygVR/aYbW5LH7rPyk+e922VYkcAAcNJOTZXFeG2
ZHPiYWgxz3QESt2ljzNsWBKaDP1AL3cptLWm3mM1iyAdWbdoSjV/6P6maQdJ3o6L
UI+1CZW9CmOfVrOaSqcR8Wwwcfy8sroLQq6xPcYkHq/xKOjNbnrrfk//BulHmMHi
6mv1lIqhRs0l3fvrM5uluVwS+tTScWrwx0hFw9ETidqv
-----END CERTIFICATE-----
</ca>

<tls-auth>
-----BEGIN OpenVPN Static key V1-----
3b78cbb21724476df2f3b7c6ef0d6e96
aed7e569ff1861bbe902e09ab9e94699
063c24fd368a46c3ad9c27710b25bad4
c3f39da58a025c9ed784a1fd8655299f
114b4a7aeb2d6763c0128c0388e4a4e1
46651ba0ac428a433269234947b48bb0
414f675644a7ad84d76ad80fcd3a6dd0
23062045e5f18e3f85db88817b51abf4
6083ea56b3ad0f29b11b4ab56c6e7987
3cea3021cb7a247850ab8c5e4d0cd69e
40e5900dead6179add058c9229991251
8d5eacda76c0e5d4db81c19137d8d268
c9cb854b54a4d3d66c92862c3aef3309
a8fa3174cb7c944d12d487d58d075688
daf6b1767e4c9670d6cf0336b378ea9b
6ccd367e6561712418af8b4268ec0e56
-----END OpenVPN Static key V1-----
</tls-auth>

key-direction 1

redirect-gateway def1
```


## Setting up a Client












## References

- kind of good guide: https://openvpn.net/index.php/open-source/documentation/howto.html
- 




