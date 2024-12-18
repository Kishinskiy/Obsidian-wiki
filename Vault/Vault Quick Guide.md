
## Создание Пользователя в Vault

```sh
vault auth enable -path=<path> userpass
```

```sh
vault write auth/<userpass:path>/users/mitchellh \ password=foo \ policies=admins
```


## Создание политик

```sh
vault policy write policy-name policy-file.hcl
```

policy-file.hcl
```hcl
# Enable Transit secrets engine
path "sys/mounts/transit" {
    capabilities = ["create", "update"]
}

# Manage Transit secrets engine keys
path "transit/keys" {
    capabilities = ["list"]
}
path "transit/keys/*" {
    capabilities = ["create", "list", "read", "update"]
}
path "transit/keys/+/config" {
    capabilities = ["create", "update"]
}

# Encrypt with any Transit secrets engine key
path "transit/encrypt/*" {
    capabilities = ["create", "update"]
}

# Decrypt with any Transit secrets engine key
path "transit/decrypt/*" {
    capabilities = ["create", "update"]
}

```
