# Users and Groups

## Create a new user with Home Directory

```text
useradd -m -d /home/<username> <username>
```

## Add user to sudoers group

```text
usermod -aG sudo <username>
```

## Set Default shell for a user

```text
usermod --shell /bin/bash <username>
```

## Delete a user and purge their /home/

Use the `-r` \(`--remove`\) option to force `userdel` to remove the user’s home directory and mail spool

```text
userdel -r <username>
```

