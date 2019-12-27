# SSH

### Naming SSH Keys

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

