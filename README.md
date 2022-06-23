# HashiCorp Vault Cookbook

- [HashiCorp Vault Cookbook](#hashicorp-vault-cookbook)
  - [Introduction](#introduction)
  - [Documentation](#documentation)


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
