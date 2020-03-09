# GPG

## GPG: Extract private key and import on different machine

After extending the expiry date of a GPG key you might have to copy your key to another machine to use the same key there. Here is how:

1. Identify your private key by running `gpg --list-secret-keys`. You need the ID of your private key \(second column\)
2. Run this command to export your key: `gpg --export-secret-keys $ID > my-private-key.asc`
3. Copy the key to the other machine \(`scp` is your friend\)
4. To import the key, run `gpg --import my-private-key.asc`

If the key already existed on the second machine, the import will fail saying 'Key already known'. You will have to delete both the private and public key first \(`gpg --delete-keys` and `gpg --delete-secret-keys`\)

