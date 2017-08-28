# Creating certificate files

Detailed instructions for creating the certificate files (required for the `worldvm2` demo) are beyond the scope of this article. However, you can generate some self-signed certificates using the following script:

```
#! /bin/bash

export HOSTNAME=`hostname -f`

openssl genrsa -out private.key 2048

openssl req -x509 -nodes -new -key private.key -days 1000 \
-out ca.crt -subj "/C=US/ST=New York/L=NYC/O=mongodb/CN=myCA"

openssl req -new -nodes -newkey rsa:2048 \
-keyout mongod.key -out mongod.csr \
-subj "/C=US/ST=New York/L=NYC/O=mongodb/CN=$HOSTNAME"

openssl x509 -CA ca.crt -CAkey private.key -CAcreateserial \
-req -days 1000 -in mongod.csr -out mongod.crt

cat mongod.key mongod.crt > mongod.pem

openssl req -new -nodes -newkey rsa:2048 \
-keyout mongosqld-client.key -out mongosqld-client.csr \
-subj "/C=US/ST=New York/L=NYC/O=mongodb/CN=mongosqld-client"

openssl x509 -CA ca.crt -CAkey private.key -CAserial ca.srl -req -days 1000 -in mongosqld-client.csr -out mongosqld-client.crt

cat mongosqld-client.key mongosqld-client.crt > mongosqld-client.pem

openssl req -new -nodes -newkey rsa:2048 \
-keyout mongosqld-server.key \
-out mongosqld-server.csr \
-subj "/C=US/ST=New York/L=NYC/O=mongodb/CN=$HOSTNAME"

openssl x509 -CA ca.crt -CAkey private.key -CAserial ca.srl -req -days 1000 -in mongosqld-server.csr -out mongosqld-server.crt

cat mongosqld-server.key mongosqld-server.crt > mongosqld-server.pem

openssl req -new -nodes \
-keyout mysql.key \
-out mysql.csr \
-subj "/C=US/ST=New York/L=NYC/O=mongodb/CN=mysql"

openssl x509 -CA ca.crt -CAkey private.key -CAserial ca.srl -req -days 1000 -in mysql.csr -out mysql.crt

openssl rsa -in mysql.key -out mysql.key
cat mysql.key mysql.crt > mysql.pem

openssl verify -CAfile ca.crt mongod.crt
openssl verify -CAfile ca.crt mongosqld-client.crt
openssl verify -CAfile ca.crt mongosqld-server.crt
openssl verify -CAfile ca.crt mysql.crt
```

The output files should be copied into a new `certs` subfolder, i.e., `.../world2/certs`
