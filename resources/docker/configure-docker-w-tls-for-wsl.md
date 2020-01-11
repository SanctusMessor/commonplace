---
description: >-
  Securing communications between Docker on Windows Subsystem Linux and Windows
  10
---

# Configure Docker w/TLS for WSL

Source: [https://raesene.github.io/blog/2018/03/29/WSL-And-Docker/](https://raesene.github.io/blog/2018/03/29/WSL-And-Docker/)  
Ref: [https://docs.docker.com/engine/security/https/\#create-a-ca-server-and-client-keys-with-openssl](https://docs.docker.com/engine/security/https/#create-a-ca-server-and-client-keys-with-openssl)

```bash
#### Server config [ Docker Daemon ]
# Docker config root in Windows
cd /mnt/c/ProgramData/Docker
mkdir certs
cd certs

# Generate Server keys and cert
openssl genrsa -aes256 -out ca-key.pem 4096
openssl req -new -x509 -days 3650 -key ca-key.pem -sha256 -out ca.pem
openssl genrsa -out server-key.pem 4096
```

> @bathindahelper I had the same issue as you on Ubuntu 18.04.x. Removing \(or commenting out\) RANDFILE = $ENV::HOME/.rnd from /etc/ssl/openssl.cnf worked for me. **Source**: [https://github.com/openssl/openssl/issues/7754](https://github.com/openssl/openssl/issues/7754)

```bash
# Create Server CSR (Certificate Signing Request)
openssl req -subj "/CN=127.0.0.1" -sha256 -new -key server-key.pem -out server.csr

# Set attributes
echo subjectAltName = IP:127.0.0.1 >> extfile.cnf
echo extendedKeyUsage = serverAuth >> extfile.cnf

# Generate signed cert
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf

#### Client Config
# Create a key
openssl genrsa -out key.pem 4096

# Create Client CSR (Certificate Signing Request)
openssl req -subj '/CN=client' -new -key key.pem -out client.csr

echo extendedKeyUsage = clientAuth >> client-extfile.cnf

# Generate signed cert
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile client-extfile.cnf

# Copy to Client
cp ca.pem ~/.docker
cp cert.pem ~/.docker
cp key.pem ~/.docker

# Set ENV variables
export DOCKER_HOST=tcp://127.0.0.1:2376
export DOCKER_TLS_VERIFY=1
```

{% hint style="info" %}
Add to ~/.zshrc  
  
export DOCKER\_HOST=tcp://127.0.0.1:2376   
export DOCKER\_TLS\_VERIFY=1
{% endhint %}

{% hint style="warning" %}
 `C:\ProgramData\Docker\config\daemon.json`
{% endhint %}

```bash
{
  "registry-mirrors": [],
  "insecure-registries": [],
  "debug": true,
  "experimental": true,
  "hosts": [
    "tcp://127.0.0.1:2376",
    "npipe://"
  ],
  "tlsverify": true,
  "tlscacert": "c:\\ProgramData\\docker\\certs\\ca.pem",
  "tlscert": "c:\\ProgramData\\docker\\certs\\server-cert.pem",
  "tlskey": "c:\\ProgramData\\docker\\certs\\server-key.pem"
}
```

## Presenting the Docker Daemon from a Ubuntu 19.10 VM in vmware to the Ubuntu Subsystem Linux in Windows 10

```bash
sudo dockerd \
  --debug \
  --tls=true \
  --tlscert=/home/username/.docker/server-cert.pem \
  --tlskey=/home/username/.docker/server-key.pem \
  --host tcp://192.168.142.128:2376
```

I was able to manually specify the above and get things running. However, I'd much rather have the config load when I start the VM.

{% hint style="danger" %}
Set a static IP. You will need to do this in the above steps when creating the CA and certs.
{% endhint %}

{% code title="/etc/docker/daemon.json" %}
```javascript
{
    "debug": true,
    "hosts": [
        "tcp://192.168.142.128:2376"
    ],
    "tls": true,
    "tlscert": "/home/username/.docker/server-cert.pem",
    "tlskey": "/home/username/.docker/server-key.pem"
}
```
{% endcode %}

You must explicitly state the startup command otherwise the default `systemctl` runner includes flags which conflict with the `daemon.json`

{% code title="/etc/systemd/system/docker.service.d/docker.conf" %}
```text
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
```
{% endcode %}



