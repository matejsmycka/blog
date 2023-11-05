# Guide: Secret management in the systems development lifecycle

This text describes different options in secret management. It is not a complete tutorial. It is collection of our experience and research.

## Main points

- Start somewhere, even if it is not perfect.

- Optimal way for a production are services like Hashicorp Vault, Azure Key Vault, etc.

- Use something which either loads the password from a file (`.env` file), or use environment variables.

- Wheter you use tool X or Y, anything is better than plaintext passwords in your git repository.

- Hardcoded secrets in the code are always bad.

- Use `ansible-vault` for deployment.

## Table of contents 

[[_TOC_]]

## Introduction

Secret management is a process of storing and distributing secrets. Secrets are passwords, API keys, certificates, etc. Anything that is sensitive and should not be publicly available. 

There is not optimal solution for secret management. Every solution has its pros and cons.
It always will be stored on HDD or in memory, so it is always possible to extract it. The goal is to make it as hard as possible.

The ideal secret management tool should be:

- Easy to use
- Easy to distribute the secrets
- Easy to rotate the secrets and access to them

Be aware that secrets might be found in the following places:

- Git history
- Logs
- Bash history

## Production instance management

There are multiple ways how to manage secrets in production. This chapter is about the most common ones. Automation should take care of the secrets, to reduce the human factor.

OWASP Secret management cheatsheet describes three good approaches[^2]:

    - Secrets pipeline: Having a secrets pipeline which does large parts of the secret management (E.g. creation, rotation, etc.)

    - Using dynamic secrets: When an application starts it could request it's database credentials, which when dynamically generated will be provided with new credentials for that session. Dynamic secrets should be used where possible to reduce the surface area of credential re-use. Should the application's database credentials be stolen, upon reboot they would be expired.

    - Automated rotation of static secrets: Key rotation is a challenging process when implemented manually, and can lead to mistakes. It is therefore better to automate the rotation of keys or at least ensure that the process is sufficiently supported by IT.

Achevieing this is not easy, however the key management services can help with this.

## Production instances

### Secret management services (SMS)

Services that can be used for secret management.
- Hashicorp Vault
- Azure Key Vault
- Cloud KMS
- AWS Secrets Manager
- Infisical
- Sops 
...

These solutions are optimal (not perfect), but without the infrastructure, for a secret management, it is not worth it to build the stack yourself.

Reason they are the optimal solution. Application uses tokens to access the secrets. You can use multi-factor authentication, and other security measures to protect the secrets. Like private key + secret token, IP whitelist, etc. You can audit access, manage, and rotate the secrets as is needed.

**Pros:**

- You can rotate the secrets without changing the token and vice versa
- You can revoke the access to the secrets
- You can audit the access to the secrets
- Compromised token does not mean compromised service right away
- Easier secret distribution and management

**Cons:**

- You still have to store the tokens somewhere
- You still have to be careful with pushing the tokens to git
- Usually, you have to pay for the service, or you have to build the infrastructure yourself
- You need SMS server and client

As an example, we will use Hashicorp Vault for storing PostgreSQL credentials.
Demo is from official documentation [^3].

```bash
# enable database secrets
vault secrets enable database
# configure database connection
vault write database/config/postgresql \
     plugin_name=postgresql-database-plugin \
     connection_url="postgresql://{{username}}:{{password}}@$POSTGRES_URL/postgres?sslmode=disable" \
     allowed_roles=readonly \
     username="root" \
     password="rootpassword"

#configure role
tee readonly.sql <<EOF
CREATE ROLE "{{name}}" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}' INHERIT;
GRANT ro TO "{{name}}";
EOF
# create role
vault write database/roles/readonly \
      db_name=postgresql \
      creation_statements=@readonly.sql \
      default_ttl=1h \
      max_ttl=24h
```
Now secrets are generated on the fly, and they are valid for 1 hour. 
They still can be retrieved, but it makes it harder to extract them. Also they cant be used right away when retrieving the token with Local File Inclusion. 

List of supported DBs: https://developer.hashicorp.com/vault/docs/secrets/databases#database-capabilities


Developer without the infrastructure is left with the following theretical options:
- Environment variables
- Files
- Something, that loads the secrets somehow directly to the memory or to the binary of the application. (Console input falls into this category) 
- Combination of the above

### Environment variables

This is the easiest way to store secrets. It is not persistent, but there are workarounds for that, like `.env` files which are loaded by the some initial process, and cannot be read afterwards. `.env` files should be ignored by git, so there is problem how to distribute them.

```bash
VAR1=secret1 VAR2=secret2 ./my_app

# Print variable
echo $VAR1
```
```powershell
$Env:password="secret12321" ./my_app

# Print variable
get-item -path Env:password | select -exp Value
```
### Files

Another nonoptimal way is to store the secrets. This approach is persistent, however it is less secure, due to Local File Inclusion (LFI) vulnerabilities. 

Loading them from something like `.env` or `config.yml` is language-specific.

Files and Env variables are not great, but still they are better than plaintext passwords in git.

### Compiled languages

Compiled languages like Go, Rust, C++ are different, because they are compiled to machine code, and it is harder to extract the secrets from the binary. However, it is still possible. Sure you can use encryption, but then you have to store the key somewhere, and it is the same problem as before. This is security by obscurity, which is not a good practise."System security should not depend on the secrecy of the implementation or its components."[^4]

## Local machine and remote git

To store secrets remotely on the git, you have to encrypt them. There are multiple ways how to do it.

### Ansible vault

Docs: https://docs.ansible.com/ansible/2.9/user_guide/vault.html

`ansible-vault` is very easy to use. It is a command line tool that encrypts and decrypts files. 

- It is integrated with Ansible to encrypt variables in playbooks on the fly. 
- You can encrypt any file, but it is most useful for encrypting variables.
- Ansible vault is included in the Ansible core.
- Vault is managed by only one password
- Deploy and source control with the `ansible-vault` is elegant, if you have a way, how to distribute the master-passwords [^1] to all developers. 


#### Tutorial

Create file **vars.yml** with content:
```yaml
---
password: heslo-je-maslo
```

Ansible-vault:
```bash
# Encrypt a file, you will be prompted for a password
ansible-vault encrypt vars.yml

# Show the encrypted file
ansible-vault view vars.yml

# Edit the encrypted file
ansible-vault edit vars.yml
```

This will create a file with the following content:
```yaml
$ANSIBLE_VAULT;1.1;AES256
373263656562383930346536323230... # encrypted content
```

Usage:
```bash
# Run ansible command
ansible localhost -m ansible.builtin.debug -a var="password" -e "@vars.yml --ask-vault-pass"

# Output
localhost | SUCCESS => {
    "password": "heslo-je-maslo"
}

# Or run playbook with pass in the file
ansible-playbook playbook.yml --vault-password-file ~/.ansible-prod-password.txt 
```

### Amber 

https://github.com/fpco/amber

Ansible vault alternative, more suitable for usage on remote machines.
Master-password is used. No external dependencies are needed. It is like pass or SecretManager for powershell, however no GPG key or user account is needed, just the master-password.

### Password managers

#### Passbolt - The Open Source Password Manager For Teams

https://janus.csirt.muni.cz 

Passbolt is WebApp that allows you to share and store credentials securely.
Passbolt has an REST API and CLI client. However, I dont recommend this tool for usage in the automation process. You cannot create group-level or object-level access tokens, only user-level permissions, which is not suitable for any other use than manual usage.

#### Pass - The Standard UNIX Password Manager

https://www.passwordstore.org/

Pass is a CLI tool based on GPG authentication. It can be integrated with git. It is very versatile. You can use multiple GPG keys for the same and different passwords. Which behaves like an `ansible-vault` however, it is not integrated with Ansible. Definitely usable, but the learning curve is higher.

##### Ansible integration
Module `community.general.passwordstore` provides a way to use pass in Ansible, same as `ansible-vault`. 
It is not included in the Ansible core.

Powershell alternative: https://github.com/PowerShell/SecretManagement

##### Usage

```bash
# Setup GPG
gpg --full-generate-key

# Generate password
pass generate <password-name> <length>

# Show password
pass <password-name>

# Ansible pass lookup
ansible localhost -m ansible.builtin.debug -a var="password" -e "password={{ lookup('community.general.passwordstore', 'heslo') }}"

# Output
localhost | SUCCESS => {
    "password": "jfKuBd0l4Nt3"
}
```

#### Gopass - The slightly more awesome standard UNIX password manager for teams 

https://www.gopass.pw/

As the name suggests, it is pass but solves some of the issues, like native multi-user support, and other usefull features. Can be used in automation. Compatible with pass storages, so you dont have to migrate your passwords.
Passwordstore [ansible](####Ansible-integration) is compatible with gopass.

##### Usage

```bash
# Spawn environment with secrets 
 gopass env <password-name> /usr/bin/bash -c "printenv <PASSWORD-NAME>"

# Output
jfKuBd0l4Nt3
```

### Zip file with password

As you propably guessed this is bad solution, but it is still better than plaintext passwords in git.

### Git-secret

https://github.com/sobolevn/git-secret

This tool encrypts files in git, using public key of the specified users.

## Secret detection

To read about secret detection, go to [secret_detection.md](https://gitlab.ics.muni.cz/525073/thesis-pub/-/blob/master/guide_secret_detection.md)

## Resources

[^1]: Master-passwords are passwords that are used to encrypt other passwords. Ansible vault masterpassword encrypts vault.

[^2]: Link to the [Cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html#24-automate-secrets-management)

[^3]: Official [documentation](https://developer.hashicorp.com/vault/tutorials/db-credentials/database-secrets) of the Hashicorp Vault

[^4]: Page 15 NIST [SP800](https://nvlpubs.nist.gov/nistpubs/legacy/sp/nistspecialpublication800-123.pdf)
