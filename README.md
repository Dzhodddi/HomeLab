# Homelab GitOps repo

Ansible configuration for a home server running Debian.
Everything on the server comes from this repo.

## What you need on your PC

- Ansible (`ansible-core`)
- The collections this repo uses:

  ```bash
  ansible-galaxy collection install -r requirements.yml
  ```

- An SSH key for Ansible:

  ```bash
  ssh-keygen -t ed25519 -C "ansible@homelab" -f ~/.ssh/ansible_homelab
  ```

## Setting up a new server

### Step 1: bootstrap

This creates the `ansible` user, installs its SSH key and gives it passwordless sudo.
The `ansible` user doesn't exist yet, so this one run logs in as your existing sudo user with your personal key:

```bash
ansible-playbook bootstrap.yml -e ansible_user=<your-user> -e ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

`-e` overrides the user and key set in the inventory for this run only.

### Step 2: check it worked

```bash
ansible all -m command -a whoami
```

This should print `ansible`.

```bash
ansible all -b -m command -a whoami
```

This should print `root`.

From now on, Ansible connects as `ansible`, and you can run playbooks without any overrides.

## Repo layout

| File | What it is |
| --- | --- |
| `ansible.cfg` | Ansible settings: inventory path, SSH pipelining |
| `inventory/hosts.yml` | The server and how to connect to it |
| `requirements.yml` | Ansible collections this repo needs |
| `bootstrap.yml` | One-time setup of the `ansible` user |