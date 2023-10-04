#tech-tip 

## Specifying SSH Key to use in 1Pass

This ssh config will make sure that this public key will be used for the specified host:L `cbre-migration-2-snowflake.pub`:

```ssh config
Host ssh.dev.azure.com
  HostName ssh.dev.azure.com
  User git
  IdentityFile ~/.ssh/cbre-migration-2-snowflake.pub
  IdentitiesOnly yes

```

Taken from [here](https://1password.community/discussion/130911/chosing-which-ssh-key-to-use).