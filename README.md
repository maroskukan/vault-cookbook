# HashiCorp Vault Cookbook

- [HashiCorp Vault Cookbook](#hashicorp-vault-cookbook)
  - [Introduction](#introduction)
  - [Documentation](#documentation)
  - [Architecture](#architecture)
    - [Components](#components)
      - [Storage Backends](#storage-backends)
      - [Secrets Engines](#secrets-engines)
      - [Authentication Methods](#authentication-methods)
      - [Audit Devices](#audit-devices)
    - [Paths](#paths)
    - [Data Protection](#data-protection)
    - [Seal and Unseal](#seal-and-unseal)
  - [Installation](#installation)
    - [Binary](#binary)
    - [Package](#package)
    - [Source](#source)
    - [Container](#container)
  - [Initialization](#initialization)
    - [Dev Server](#dev-server)
  - [Configuration](#configuration)
    - [Secrets Engines](#secrets-engines-1)
    - [Authentication](#authentication)
    - [Policies](#policies)


## Introduction

HashiCorp Vault is a software solution that provides lifecycle management (create, generate, update, delete) and access to secrets (Usernames, Passwords, API keys, Certificates, Encryption Keys) across the entire origanization.

Vault provides the following benefits to organization:
- Store Long-Lived, Static Secrets
- Dynamically Generate Secrets, upon request
- Identity-Based Access Across different Clouds and Systems
- Provides Encryption as a Service
- Act as a Root or Intermediate CA

Vault addresses the following use cases within the organization:
- Centralized storage of secrets
- Migration of dynamically generated secrets
- Secured data with centralized workflow for Encryption operations
- Automation of X.509 certificate generation
- Migration to identity based access

Vault provides multiple interfaces for interaction:
- API (Machines)
- UI (Humans)
- CLI (Machines or Humans)

A typical secret retrieval workflow would include an authenticate through one of methods (Username and Password, RoleID and SecretID, TLS Certificate, Integrated Cloud Credentials) to one of the interfaces (UI,API,CLI). Afterwards a token which will be valid for a certain duration (TTL) is received and would be used to access various areas (Vault Path(s) with the correct permission Read/Write/Delte/List).

Vault provides the following deployment options:
- Self-Hosted and Managed
  - Open Source
  - Enterprise
- HashiCorp Hosted & Managed
  - Vault on HCP


## Documentation

- [Vault Project](https://www.vaultproject.io/)
- [HashiCorp Cloud Platform](https://cloud.hashicorp.com/)


## Architecture

### Components

Vault core components include:
- Storage Backends
- Secrets Engines
- Authentication Methods
- Audit Devices

#### Storage Backends

- Configures location for sotrage of Vault data (in-memory, local storage, dynamoDB, Consul, S3)
- Storage is defined in the main Vault configuration file with desired parameters
- All data is encrypted in transit and at-rest using AES256
- Nt all storage backends are created equal:
  - Some support HA
  - Others have better tools for management and data protection
- There is only one storage backend per Vault cluster

#### Secrets Engines

- Responsible for managing secrets
- Can store, generate or encrypt data
- Can connect to other services to generate dynamic credentials on-demand
- Can be bevaled and used as needed:
  - Even multiple secrets engines of the same type
- Enabled and isolated at a *path*
  - All interactions are done directly with the *path* itself

#### Authentication Methods

- Perform authentication and manage idendities
- Responsible for assigning identity and policy to user
- Multiple methods can be enabled for different entities (machine vs human)
- Once authenticated, Vault will issue a client token used in subsequend requests
- Each token has associated policy and TTL
- Default authentication method for a new Vault deployment is token

#### Audit Devices

- Keep detailed log of all requests and responses
- Formatted using JSON
- Sensitive information is hashed
- Should have more than one audit device enabled
  - Vault requires at least one audit device to write log before completing Vault request if enabled

### Paths

- Everything in Vault is path-based
- The path prefix tells Vault which component a request should be routed
- Secret engines, authentication methods, and audit devices are mounted at a specified path, often referred to as a mount
- Paths available are dependend on the features enabled in Vault, such as Auth Methods and Secrets Engines
- System backend is a default backend in Vault which is mounted at the /sys endpoint
- Vault components can be enabled at ANY path using the `-path` flag. Each component does have a default path you can use as well
- Vault has a few System Reserved Path which you cannot use or remove:

  | Path Mount Point | Description                                                         |
  | ---------------- | ------------------------------------------------------------------- |
  | `auth/`          | Endpoint for auth method configuration                              |
  | `cobbyhole/`     | Endpoint used by the Cubbyhole secrets engine                       |
  | `identity/`      | Endpoint for configuring Vault identity (entities and groups)       |
  | `secret/`        | Endpoint used by Key/Value v2 secrets engine if running in dev mode |
  | `sys/`           | System endpoint for configuring Vault                               |

### Data Protection

Master Key - used to decrypt the encryption key
- Created during Vault initialization or during a rekey operation
- Never written to storage when using traditional unseal mechanism
- Written to core/master (storage backend) when using Auto Unseal

Encryption Key - used to encrypt/decrypt data written to storage backend
- Encrypted by the Master Key
- Stored alongside the data in a keyring on the storage backend
- Can be easily rotated (manual operation)

### Seal and Unseal

- Vault starts in a sealed state, meaning it knows where to access the data, and how, but can't decrypt it
- Almost no operations are possible when Vault is in a sealed state (only status check and unsealing are possible)
- Unsealing vault means that a node can reconstruct the master key in order to decrypt the encryption key and read the data
- After unsealing, the encryption key is stored in memory
- Sealing means that Vault discard the encryption key and requires unseal to perform further operations
- Vault will start in a sealed state and you can also manually seal it using UI, CLI or API
- You should seal Vault when:
  - Key shareds are inadvertently exposed
  - Detection of a compromise or network intrustion
  - Malware on the Vault nodes


## Installation

Vault is written in Go language with source code available on Github. It is distributed as single binary but other options such as repository package or container image are available.

### Binary

One of the most straighforward methods. Simple download the zip file for approripate architecture and place it somwhere in the exection path.

```bash
wget -q https://releases.hashicorp.com/vault/1.11.0/vault_1.11.0_linux_amd64.zip

wget -qO- https://releases.hashicorp.com/vault/1.11.0/vault_1.11.0_SHA256SUMS \
     | grep vault_1.11.0_linux_amd64.zip \
     | sha256sum -c -
vault_1.11.0_linux_amd64.zip: OK

unzip vault_*

./vault version
Vault v1.11.0 (ea296ccf58507b25051bc0597379c467046eb2f1), built 2022-06-17T15:48:44Z
```

### Package

Another method is to leverage a packege manager such as `Homebrew`, `Chocolatey`, `Aptitude` to download, install and upgrade the binary. The benefit of this approach is that this method will also create basic service definition (`vault.service`) and configuration file (`vault.hcl`).

### Source

In order to compile from source, you need to have Go available locally or you can use a Docker image that contains Go and execute compilation within this container.

```
# Clone repo, enter container and compile
git clone git@github.com:hashicorp/vault
cd vault
docker run --rm -it -v "$PWD":/usr/src/myapp -w /usr/src/myapp golang bash
make bootstrap
make dev

# Verify the produced binary
./bin/vault version
Vault v1.12.0-dev1 ('2ccc3e0e6be4340ec69421b05d45e67751d46891'), built 2022-06-28T19:54:24Z
```

### Container

Vault server can be started as container with the following options:

```bash
docker container run -d \
                     -p 8200:8200 \
                     --cap-add=IPC_LOCK \
                     --name=dev-vault \
                     vault
```

This will start the container in background with a local port mapping. The generated Unseal Key and Root Token can be displayed by retrieving container logs.

```bash
docker container logs dev-vault 2>&1 | grep 'Unseal Key\|Root Token'
Unseal Key: xZ9QJzJIYE6yFkq4Z/mi4C8hzyvPqG9j1KuBT7kT24U=
Root Token: hvs.VB6yvJb8xI0h2tGxZWM1sw97
```

It may happen that the container is logging into `stderr` therefore in order for pipe to work, redirect the output to `stdout` first.


## Initialization

After you download and install binary, it is now time to initialize the vault server. The procedure (starting and unsealing) will be different depending on use case (Development vs. Production).

### Dev Server

Dev server is used, as name implies for development and testing, it should never be used in production. It automatically starts in unsealed state, prints out the `Unseal Key` and `Root Token` in output. It uses in-memory storage backend, therefore it does not persist secret data.

It can be started with the `-dev` argument.

```bash
# Start in dev mode in background
vault server -dev &

# Update VAULT_ADDR and Verify status
export VAULT_ADDR='http://127.0.0.1:8200'
vault status
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.10.4
Storage Type    inmem
Cluster Name    vault-cluster-f366bb73
Cluster ID      c313998c-10c8-5077-d3e4-12137cd04cab
HA Enabled      false
```

To Write a key/value pair, interact with `kv` Key-Value storage and use `put` method against the default kv secret storage path at `secret/`.

```bash
vault kv put secret/mySecret password=myPassword
==== Secret Path ====
secret/data/mySecret

======= Metadata =======
Key                Value
---                -----
created_time       2022-07-04T12:36:28.3963921Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1
```

To retrieve the secret use `get` method.

```bash
vault kv get secret/mySecret
==== Secret Path ====
secret/data/mySecret

======= Metadata =======
Key                Value
---                -----
created_time       2022-07-04T12:36:28.3963921Z
custom_metadata    <nil>
deletion_time      n/a
destroyed          false
version            1

====== Data ======
Key         Value
---         -----
password    myPassword
```

To retrieve in json format and pasrse with `jq` utlity.

```bash
# Using jq
vault kv get -format=json secret/mySecret | jq -r '.data.data.password'

# Using field option
vault kv get -field=password secret/mySecret
```


## Configuration

### Secrets Engines

Built-in secrets enginers include:
- Kye/Value (KV)
- Cloud (Azure, AWS, GCP)
- GitHub
- Cubbyhole
- SSH
- Database
- Token
- Nomad
- And more...

When enabled, a secrets engine is available at path that acts as a virtual file system that is mounted. An example of such paths include:

- secret (secret/mySecret/dbPassword)
- ssh (ssh/roles/admin)

```bash
# List existing (default) secrets engines
vault secrets list
Path          Type         Accessor              Description
----          ----         --------              -----------
cubbyhole/    cubbyhole    cubbyhole_484a7c75    per-token private secret storage
identity/     identity     identity_0d47327e     identity store
secret/       kv           kv_35d3194f           key/value secret storage
sys/          system       system_d1bf4ea6       system endpoints used for control, policy and debugging

# Enable a kv secrets engine at custom path
vault secrets enable -path=myApp kv

# Write a secret to custom path
vault kv put myApp/myOtherSecret key=value

# Read a secret
vault kv get myApp/myOtherSecret
=== Data ===
Key    Value
---    -----
key    value
```

```
# Enable a database secrets engine at default path (database/)
vault secrets enable database

# Write to a default path
vault write database/config/mysql_app1

# Mounts a database secret engine to a custom path
vault secrets enable -path=myAppDB database

# Write to a custom path
vault write myAppDB/config/mysql_app1
```

### Authentication

Vault provides the following authentification methods for users:
- Userpass
- Active Directory and LDAP
- Cloud providers
- GitHub

Vault provides the following authentification methods for machines:
- AppRole
- Kubernetes

Examples:

```bash
# List existing authentication methods
vault auth list
Path      Type     Accessor               Description
----      ----     --------               -----------
token/    token    auth_token_1419cf7c    token based credentials

# Enable userpass authentication method
vault auth enable userpass

# Create user
vault write auth/userpass/users/vaultuser password=vault

# Verify that the user can login (a token will be returned back)
vault login -method=userpass username=vaultuser password=vault

# Verify that the user can login with token directly
vault login hvs.CAESIJvoWvslZPyxaT7TDG8aGc8FKrjKfqfDNKwAZTwMuPkrGh4KHGh2cy5IWUxjRTRRaFBHeEkyMVYyMVh4bERFU1M

# Create a new child token with same privileges as parent
vault token create

# List token accessors
vault list auth/token/accessors

# View more information about token
vault token lookup -accessor vqYYe6au3XZxJkOReQMFTlLP

# Revoke a token
vault token revoke -accessor vqYYe6au3XZxJkOReQMFTlLP

# Create a new child token with TTL
vault token create --ttl=5m
```

### Policies

Policies perform authorization against authentificated request. They are written in HCL and associated to tokens when generated.

Built-in policies include:
- `root` which provides sudo access to all paths, including `sys`. The only path a root token cannot access is a cubbyhole.
- `default` which is attached to all tokens, allows read its metadata and can be modified

Example policies

```ruby
path "secret/dev/*" {
  capabilities = ["create", "update", "read", "list"]
}
```

```bash
# List existing policies
vault policy list
default
root

# View default policy
vault policy read default

# Upload policies
vault policy write dev-policy ./examples/policies/dev-policy.hcl
vault policy write app-policy ./examples/policies/app-policy.hcl

# Enable userpass auth method
vault auth enable userpass

# Create app and dev user
vault write auth/userpass/users/app password=app policies=app-policy
vault write auth/userpass/users/dev password=dev policies=dev-policy

# Login as dev user
vault login -method=userpass username=dev password=dev

# Verify which methods are allowed against a path
vault token capabilities secret/data/dev/

# Create a new secret
vault kv put secret/dev/appsecret user=dbUser
```