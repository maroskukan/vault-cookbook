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
