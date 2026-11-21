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

This creates the `ansible` user, installs its SSH key, gives it passwordless sudo, and installs `python3-apt` (Ansible's `apt` module needs it, also for `--check`).
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

### Step 3: apply the configuration

Preview what would change, without changing anything:

```bash
ansible-playbook site.yml --check --diff
```

Apply it:

```bash
ansible-playbook site.yml
```

Run it again: it should finish with `changed=0`. That means the server already matches this repo.

`site.yml` is the main playbook. Run it after every change to the repo.

## What gets configured

### `base` role

- Installs the packages listed in `roles/base/defaults/main.yml`
- Turns on automatic security updates with `unattended-upgrades`
- Unmasks and starts the `apt-daily` timers, which some images (such as DietPi) mask by default

Check the timers on the server:

```bash
ansible all -m command -a 'systemctl list-timers apt-daily*'
```

## Repo layout

| File | What it is |
| --- | --- |
| `ansible.cfg` | Ansible settings: inventory path, SSH pipelining |
| `inventory/hosts.yml` | The server and how to connect to it |
| `requirements.yml` | Ansible collections this repo needs |
| `bootstrap.yml` | One-time setup of the `ansible` user |
| `site.yml` | Main playbook: applies all roles to the server |
| `roles/base/` | Packages and automatic updates |
