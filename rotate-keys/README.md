# SSH Automatic Key Rotation Playbook

## Overview

This Ansible playbook automates the rotation of SSH Ed25519 keys across your infrastructure every 90 days. It generates new key pairs, distributes them to remote servers, tests connectivity, removes old rotation keys, and maintains an audit trail. 

## Usage

```bash
ansible-playbook rotate-keys.yml -i inventory.ini
```
