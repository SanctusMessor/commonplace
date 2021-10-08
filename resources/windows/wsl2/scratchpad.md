---
description: /etc/wsl.conf
---

# Scratchpad

## [https://docs.microsoft.com/en-us/windows/wsl/wsl-config](https://docs.microsoft.com/en-us/windows/wsl/wsl-config)

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

## Remove Windows Path from WSL

```text
[interop]
appendWindowsPath=false
```

## Start WSL2 as a specific user

```text
wsl -d <distro> -u <user>
```

## Notes when creating from rootfs

Create a non-root user

```bash
# in Ubuntu, you'll be root by default
adduser username
```

