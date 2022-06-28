# Vault on Ubuntu 22.04

## Provisioning - Option A


```bash
# Build and provision all at once
vagrant up

# Verify vault status
vagrant ssh -c 'vault status'
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.11.0
Build Date      2022-06-17T15:48:44Z
Storage Type    file
Cluster Name    vault-cluster-53f52a4a
Cluster ID      847a31e8-7dc0-a0cf-6926-0d97b10d265a
HA Enabled      false

# Cleanup
vagrant destroy --force
rm -rf init_root init_unseal
```

## Provisioning - Option B

```bash
# Build and provision separately
vagrant up --no-provision

vagrant provision --provision-with install
vagrant provision --provision-with init
vagrant provision --provision-with unseal

# Verify vault status
vagrant ssh -c 'vault status'
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.11.0
Build Date      2022-06-17T15:48:44Z
Storage Type    file
Cluster Name    vault-cluster-53f52a4a
Cluster ID      847a31e8-7dc0-a0cf-6926-0d97b10d265a
HA Enabled      false

# Cleanup
vagrant destroy --force
rm -rf init_root init_unseal
```
