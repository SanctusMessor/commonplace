# Users and Groups

## Create a new user with Home Directory

```bash
useradd -m -d /home/<username> <username>
```

## Add user to sudoers group

```bash
usermod -aG sudo <username>
```

## Set Default shell for a user

```bash
usermod --shell /bin/bash <username>
# OR #
chsh --shell /bin/sh <username>
```

## Delete a user and purge their /home/

Use the `-r` \(`--remove`\) option to force `userdel` to remove the user’s home directory and mail spool

```bash
userdel -r <username>
```

