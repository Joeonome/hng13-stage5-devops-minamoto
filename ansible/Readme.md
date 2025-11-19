# Configuration Management with Ansible

This repository contains the full automation setup for deploying Legalwatchdog application across staging and production environments. The goal of this project is to make deployments consistent, secure, and fully automated.

This repo is built around Ansible, and it manages the entire lifecycle of both the frontend and backend for the application.

## **What This Project Does**

1. **Environment-Aware Deployments**

The automation can deploy to staging or production without you passing any extra flags.
Ansible detects which inventory group a server belongs to and applies the correct:

- Environment variables

- App configuration

- Branches to deploy

- Domain names

- Logging setup

This helps us keep staging and production separate, reliable, and predictable.

2. **Server Provisioning & Dependencies**

The playbook installs and configures everything the applications need to run:

- Nginx

- Node.js

- Python + pip

- PM2 and systemd

- Git

- Certbot

- CloudWatch Agent

For private repositories, authentication is handled securely using Vault-encrypted SSH keys or SSH agent forwarding.

3. **Secrets Management (.env Files)**

No secrets are stored in GitHub.
All environment variables live inside a local secrets/ directory on the Ansible controller:

```
secrets/
  staging.env
  prod.env
```

During deployment, Ansible:

- Detects the server’s environment

- Picks the correct env file

- Copies it to the right application directory

- Renames it to .env

This ensures secrets stay out of version control and are deployed consistently.

4. **Application Process Management**

The frontend and backend are managed differently depending on the technology used:

- Node.js apps runs on the frontend which runs under PM2, which automatically restores processes after reboot.

- Python apps runs on the backend which runs under systemd, with services generated from templates.

With this applications restart automatically and stay running without manual intervention.

5. **Nginx Configuration**

No static config files are committed.
Nginx configurations are built using Jinja2 templates and rendered during deployment.

The templates dynamically set:

- Server names

- Reverse-proxy ports

- SSL certificate paths

- Log file locations

This ensures consistency across environments and removes hand-written configuration errors.

6. **SSL Certificates**

SSL is fully automated using Certbot, including:

- Installation

- Certificate issuance

- Renewal

- HTTPS redirection

All domains for both frontend and backend are provisioned automatically.

7. **Centralized Logging with AWS CloudWatch**

Developers don’t SSH into servers, so logs must be accessible centrally.

This repo configures the AWS CloudWatch Unified Agent to stream:

- Application logs (stdout/stderr)

- Nginx access logs

- Nginx error logs

Every log group follows a strict naming pattern required by the project:

```
/ec2/{environment}/minamoto/{component}
```

## **Repository Structure (Important Folders)**

```
ansible/
├─ inventory.ini
├─ site.yml
├─ group_vars/
│  ├─ staging.yml
│  ├─ prod.yml
│  └─ all/
│     └─ vault.yml            # contains encrypted deploy_key and other secrets (created with ansible-vault)
├─ roles/
│  └─ universal/
│     ├─ tasks/
│     │  └─ main.yml
│     ├─ templates/
│     │  ├─ nginx_site.j2
│     │  ├─ pm2_ecosystem.j2
│     │  ├─ systemd_service.j2
│     │  └─ cloudwatch-agent.json.j2
│     └─ files/
│        └─ (empty)
├─ secrets/                   # MUST be in .gitignore
│  ├─ staging.env
│  └─ prod.env
└─ .gitignore

```

- inventory.ini – defines staging and production servers

- group_vars/ – contains environment-specific variables

- roles/universal/ – the actual automation logic (tasks + templates)

- secrets/ – stores .env files and (optionally) deploy keys, but is ignored in Git

## **How Deployments Work**

Add/update code in the backend or frontend repo.

Run:

```
ansible-playbook -i inventory.ini site.yml
```

The playbook:

- Detects environment

- Installs dependencies

- Clones the correct branch

- Injects secrets

- Configures processes

- Configures Nginx

- Obtains/renews SSL certs

- Sets up CloudWatch logging

Deployments become repeatable, audit-friendly, and consistent.

## **What This Repo Is For**

This repository is specifically meant for:

- Automated provisioning of Minamoto servers (staging + prod)

- Zero-touch deployments

- Secure secrets handling

- Consistent logging and observability

- Reproducible Nginx and application configurations

It is not intended to store application code—only infrastructure and automation logic.

## **Requirements**

- Ansible (controller machine)

- AWS IAM access or EC2 instance roles for CloudWatch logging

- Valid domains pointing to the servers for SSL provisioning

## **Contributing**

If you’re making changes to the automation:

- Keep templates clean and reusable

- Update group_vars when adding new components

- Never commit anything inside secrets/

- Follow the existing folder structure for new roles or tasks
