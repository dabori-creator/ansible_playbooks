# Password policy

This playbook checks the expiration date of all users' passwords on servers from inventory.yml. If the password expires in less than 5 days, the script generates a new one, sends it to the local machine, and saves it to a file that is encrypted using gpg. Additionally, a backup of the target file is implemented to prevent loss of access.

To start it use this command:

```bash
ansible-playbook pass.yml -i inventory.yml
```
