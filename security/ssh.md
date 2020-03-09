# SSH

Secure Shell \(SSH\) is a cryptographic network protocol for operating network services securely over an unsecured network. Typical applications include remote command-line, login, and remote command execution, but any network service can be secured with SSH. [Wikipedia](https://en.wikipedia.org/wiki/Secure_Shell)

## Generating an SSH Key <a id="generating-an-ssh-key"></a>

```bash
ssh-keygen -a 1000 -t ed25519 -C "{{ YOUR_EMAIL }}"

# Alternatively
ssh-keygen -t rsa -b 4096 -C "{{ UNIQUE_NAME }}"
```

* `-a 1000` tells the key generator to use 1000 key derivation rounds when setting the passphrase, this will increase the amount of time required to brute force the password protecting the SSH key
* `-t ed25519` tells the key generator to use the `ed25519` key type
* `-C "{{ YOUR_EMAIL }}"` sets the key comment, allowing you to identify your

  public key in a list of public keys more easily \(and to remember which key

  this is\)

## Naming SSH Keys

```yaml
id_<key_algorithm>_<servername>_<purpose>
id_<key_algorithm>_<service>_<purpose>
```

With the following rules:

* If it's not for a specific server/service, remove `<servername>/<service>`
* If it's not for a specific purpose, remove `<purpose>`
* At least one of the information-types \(`<purpose>` or `<servername>/<service>`\) has to be contained in the name

Examples:

* id\_rsa\_github\_username
* id\_rsa\_server01\_rsync

## Adding a Public Key to a Remote Server

```yaml
cat ~/.ssh/id_rsa_name.pub | ssh user@hostname 'cat >> .ssh/authorized_keys'
```

## SSH config

```yaml
Host example-bastion
  User ec2-user
  Hostname ec2-100-100-100-100.compute-1.amazonaws.com
  Port 22

Host example-mongo
  User ec2-user
  Hostname 200.200.200.200
  Port 22
  ProxyCommand ssh -q -W %h:%p example-bastion
```

## Adding your SSH key to the ssh-agent

1. Ensure the ssh-agent is running:

   ```text
   # start the ssh-agent in the background
   $ eval $(ssh-agent -s)
   > Agent pid 59566
   ```

2. Add your SSH private key to the ssh-agent. If you created your key with a different name, or if you are adding an existing key that has a different name, replace _id\_rsa_ in the command with the name of your private key file.

   ```text
   $ ssh-add ~/.ssh/id_rsa
   ```

