# Documentation for Creating CA and TLS certificates

## Reference Documentation
https://jamielinux.com/docs/openssl-certificate-authority/appendix/root-configuration-file.html

https://docs.joshuatz.com/cheatsheets/security/self-signed-ssl-certs/


## Generate ca.key and ca.pem

```
touch index.txt
echo 1000 > serial

openssl genrsa -out ca.key 4096

chmod 400 ca.key

openssl req -config ca.conf \
      -key ca.key \
      -new -x509 -days 7300 -sha256 -extensions v3_ca \
      -out ca.pem

chmod 444 ca.pem
```
The CA pem can be renamed to `ca.crt` if needed.  For instance if it ends in `.crt` windows allows double click to install the cert.

If you want a passphrase for the key, you can use this command when generating the key:
```
openssl genrsa -aes256 -out ca.key.pem 4096
```

## Verify the CA
```
openssl x509 -noout -text -in ca.pem
```

## Generate a server TLS keypair
```
openssl genrsa -out server.key 2048 

chmod 400 server.key

openssl req -new -key server.key -out server.csr -config server.conf -sha256 -extensions x509_extensions

openssl x509 -req -in server.csr -CA ca.pem -CAkey ca.key \
    -CAcreateserial -out server.crt -days 365 -sha256 -extfile server.conf -extensions x509_extensions
```

If you want a passphrase for the key, you can use this command when generating the key:
```
openssl genrsa -aes256 -out server.key 2048
```

## Verify the TLS certificate
```
openssl x509 -text -noout -in server.crt

openssl verify -CAfile ca.pem server.crt
```

