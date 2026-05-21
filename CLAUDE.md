# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the Ansible infrastructure repository for the Swiss Library for the Blind, Visually Impaired and Print Disabled (SBS). It manages multiple application servers using Ansible playbooks and roles, deploying several Java-based applications: Daisyproducer2, Kati (catalog), Madras2, and Pipeline2.

**Key Principle**: This is a deployment-only repository with no source code. It orchestrates the installation and configuration of applications hosted in separate GitHub repositories under the `sbsdev` organization.

## Project Structure

```
sbs-infrastructure/
├── *.yml                    # Top-level playbooks for each application/system
├── group_vars/all.yml       # Vault-encrypted global secrets
├── host_vars/              # Host-specific variables and secrets
├── roles/                  # Ansible roles (main deployment logic)
├── files/                  # Static files (legacy .deb packages)
└── [inventory files]       # test, production, pairing (host definitions)
```

### Inventory Files

- **test**: Test server(s) for staging deployments
- **production**: Production servers
- **pairing**: Development/pairing droplet for team collaboration
- Each inventory file contains groups like `[daisyproducer]`, `[kati]`, `[madras2]`, `[prometheus]`

### Roles Directory

19 roles implement the deployment logic:

**Core Application Roles**:
- `daisyproducer2` - Document production system (depends on pipeline, java, liblouis, hyphenator, nginx)
- `kati` - Catalog application (depends on java, nginx)
- `madras2` - Audio/multimedia production (depends on java, nginx)
- `pipeline2` - DAISY Pipeline2 processing engine
- `pipeline1` - Legacy pipeline (older version)

**Supporting/Infrastructure Roles**:
- `java`, `java8`, `adoptium` - Java runtime environments
- `nginx` - Web server/reverse proxy
- `liblouis` - Braille translation library (compiled from source)
- `hyphenator` - Text hyphenation
- `dtbook2sbsform` - DTBook to SBS format converter
- `dtbook-schema` - DTBook schema definitions
- `dtbook-catalog` - XML catalog system
- `prometheus-server`, `prometheus-node` - Monitoring infrastructure

### Role Structure

Each role follows standard Ansible layout:
- `tasks/main.yml` - Primary task definitions
- `defaults/main.yml` - Default variables (versioning, paths, configuration)
- `handlers/main.yml` - Service restart handlers
- `templates/` - Jinja2 templates for config files (typically `.j2` extension)
- `meta/main.yml` - Role dependencies (using `dependencies:` list)
- `files/` - Static file assets

### Host Variables

Per-host configuration in `host_vars/<hostname>/`:
- `vars.yml` - Application-specific settings (version overrides, hostnames, paths)
- `secrets.yml` - Vault-encrypted secrets (database credentials, API keys)

Global secrets stored in `group_vars/all.yml` (vault-encrypted).

## Deployment Commands

All deployments require the vault password file (`.vault_pass.txt`). Use `-K` flag for sudo password when needed.

### Deploy to Test
```bash
ansible-playbook -i test -K --vault-password-file .vault_pass.txt <playbook>.yml
```

### Deploy to Production
```bash
ansible-playbook -i production -K --vault-password-file .vault_pass.txt <playbook>.yml
```

### Application-Specific Playbooks

- `daisyproducer2.yml` - Deploy Daisyproducer2
- `kati.yml` - Deploy Kati (requires manual font installation via play tasks)
- `madras2.yml` - Deploy Madras2
- `prometheus.yml` - Deploy Prometheus monitoring
- `pairing.yml` - Setup pairing development droplet

Each playbook applies one or more roles to targeted hosts and may include inline tasks for fonts or other assets.

## Version Management

Application versions are stored in role `defaults/main.yml` and can be overridden in `host_vars/<hostname>/vars.yml`. The full deployment cycle for Clojure apps (Daisyproducer2, Kati, Madras2):

1. Run `lein release` in the upstream repo — this triggers a GitHub Action that builds the uberjar and uploads it to GitHub Releases
2. Update the version variable in `roles/<app>/defaults/main.yml`
3. Re-run the playbook to download and deploy the new JAR from GitHub Releases

Most roles download pre-built artifacts (JAR files, .deb packages) from GitHub Releases using the `get_url` Ansible module.

## Role Dependencies

Roles declare dependencies via `meta/main.yml`. For example, `daisyproducer2` depends on:
```yaml
dependencies:
  - java
  - liblouis
  - pipeline1
  - pipeline2
  - hyphenator
  - prometheus-node
  - nginx
  - dtbook-schema
  - dtbook2sbsform
  - dtbook-catalog
```

Ansible automatically applies dependencies before the dependent role.

## Configuration Strategy

**Defaults**: Role `defaults/main.yml` contains all configurable parameters (paths, versions, mail settings, etc.)

**Templates**: Application config files (systemd services, config.edn for Clojure apps, nginx sites) use Jinja2 templates that interpolate variables.

**Handlers**: Notify handlers to restart services when configs change. Each role that manages services has `handlers/main.yml` defining restart handlers.

**Example flow**:
1. Role creates/updates a config file via template → notifies handler
2. Handler restarts the service
3. Service uses new configuration immediately (if properly implemented)

## Secrets Management

Vault-encrypted YAML files for sensitive data:

- `group_vars/all.yml` - Global secrets (used by default in all roles)
- `host_vars/<hostname>/secrets.yml` - Host-specific secrets

Edit secrets with:
```bash
ansible-vault edit --vault-password-file .vault_pass.txt group_vars/all.yml
ansible-vault edit --vault-password-file .vault_pass.txt host_vars/<hostname>/secrets.yml
```

Secrets are referenced in templates and variables like any other variable (e.g., `{{ mail_recipients }}`).

## Key Patterns and Conventions

### JAR-Based Apps
Daisyproducer2, Kati, and Madras2 follow a common pattern:
1. Download JAR from GitHub Releases
2. Create version-independent symlink (e.g., `kati.jar` → `kati-0.43.0.jar`)
3. Create config directory and render Jinja2 config templates
4. Create systemd service file via template
5. Enable and start service
6. Configure nginx reverse proxy with template

### Systemd Services
Services managed via:
- Template-rendered `.service` files in `/etc/systemd/system/`
- `systemd` module to enable and start
- Handlers to trigger daemon_reload and service restart

### Nginx Integration
All web apps (Daisyproducer2, Kati, Madras2, Pipeline2, DTBook Catalog) have:
- Nginx site-available template in role `templates/`
- Symlink from sites-available to sites-enabled
- Removal of default site
- Handler to restart nginx on config changes

## Testing and Validation

Currently no formal test suite exists. Validation is performed by:
1. Running playbooks against test inventory
2. Manual verification of deployed services
3. Git history review (inspect recent commits in `git log`)

Check recent deployment commits:
```bash
git log --oneline | head -20
```

## Key External Dependencies

All applications are built and released separately in their own GitHub repositories:
- `sbsdev/daisyproducer2`
- `sbsdev/catalog` (Kati)
- `sbsdev/mdr2` (Madras2)
- `sbsdev/pipeline` (SBS-customized modules)
- `daisy/pipeline-assembly` (official DAISY Pipeline2)
- `sbsdev/dtbook2sbsform`, `dtbook-schema`, `dtbook-catalog`, etc.

These are referenced by GitHub Releases URL in roles. Updating versions requires:
1. Creating a release in the upstream repository
2. Updating the version variable in this repository
3. Re-running the deployment

## Deprecated/Legacy Elements

- Old `.deb` files in `files/` directory are legacy and not actively used
- `java8-old` role is deprecated (replaced by `adoptium`)
- Commented-out sections in some roles (e.g., NFS mount tasks in madras2) indicate planned features
- System V init scripts were replaced with systemd services

## Repository Metadata

- **License**: MIT
- **Organization**: sbsdev (GitHub)
- **Purpose**: Infrastructure automation for accessibility document production
- **Primary Users**: SBS system administrators
- **Git Remote**: git@github.com:sbsdev/sbs-infrastructure.git
