# Build EKL with security
1. Create private key <br>
openssl genrsa -out ca.key 2048

2. Create request with private key. Need expiration <br>
openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt

3. Create server key/certificate with configuration

File server.conf
```
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
countryName                     = XX
stateOrProvinceName             = XXXXXX
localityName                    = XXXXXX
postalCode                      = XXXXXX
organizationName                = XXXXXX
organizationalUnitName          = XXXXXX
commonName                      = XXXXXX
emailAddress                    = XXXXXX

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = DOMAIN_1
DNS.2 = DOMAIN_2
DNS.3 = DOMAIN_3
DNS.4 = DOMAIN_4

[ v3_ca ]
subjectAltName = IP: 192.168.1.104
```

```
openssl genrsa -out server.key 2048
openssl req -sha512 -new -key server.key -out server.csr -config server.conf
echo "C2E9862A0DA8E970" > serial
openssl x509 -days 3650 -req -sha512 -in server.csr -CAserial serial -CA ca.crt -CAkey ca.key -out server.crt -extensions v3_req -extfile server.conf
mv server.key server.key.pem && openssl pkcs8 -in server.key.pem -topk8 -nocrypt -out server.key
```

Run server for example
```
./bin/logstash -f ./logstash.conf 
```

4. Create client key/certificate
File client.conf.
```
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no
 
[req_distinguished_name]
countryName                     = XX
stateOrProvinceName             = XXXXXX
localityName                    = XXXXXX
postalCode                      = XXXXXX
organizationName                = XXXXXX
organizationalUnitName          = XXXXXX
commonName                      = XXXXXX
emailAddress                    = XXXXXX

[ usr_cert ]
#Extensions for server certificates (`man x509v3_config`).
basicConstraints = CA:FALSE
nsCertType = client, server
nsComment = "OpenSSL FileBeat Server / Client Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment, keyAgreement, nonRepudiation
extendedKeyUsage = serverAuth, clientAuth

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth, clientAuth

[ v3_ca ]
subjectAltName = IP: 192.168.1.104
```
```
openssl genrsa -out client.key 2048
openssl req -sha512 -new -key client.key -out client.csr -config client.conf
openssl x509 -days 3650 -req -sha512 -in client.csr -CAserial serial -CA ca.crt -CAkey ca.key -out client.crt -extensions v3_req -extensions usr_cert  -extfile client.conf
```
Run client for example
```
filebeat -c ./filebeat.yml
```

```
 #If the client key is not encrypted by passphrase, it can always be added later (filebeat "ssl.key_passphrase").
 #openssl rsa -des -in client.key -out client4.key
```
