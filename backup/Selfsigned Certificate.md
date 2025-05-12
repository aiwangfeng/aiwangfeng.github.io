# Create CA key

```
openssl genrsa -aes256 -out ca.key 4096
```

# Generate a CA certificate

```
openssl req -new -x509 -sha256 -days 3650 -key ca.key -out ca.crt
```

# Create a private key for a certificate

```
openssl genrsa -out siteprivate.key 4096
```

# Create a CSR

```
openssl req -new -sha256 -subj "/CN=www.example.com" -key siteprivate.key -out site.csr
```

# Create a EXT config

```
echo "subjectAltName=DNS:*.example.com,IP:192.168.1.100" > ext.cnf
```

# Generate a site certificate

```
openssl x509 -req -sha256 -days 3650 -CA ca.crt -CAkey ca.key -in site.csr -out site.crt -extfile ext.cnf -CAcreateserial
```
