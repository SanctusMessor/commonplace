---
description: /etc/wsl.conf
---

# Scratchpad

## Set the default user for a WSL2 instance

```text
[user]
default=<string>
```

## Enable localhostForwarding

```text
[wsl2]
localhostForwarding=true
```

## Start WSL2 as a specific user

```text
wsl -d <distro> -u <user>
```

