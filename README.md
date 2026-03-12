## Purpose
`manage_users.yml` can create or delete accounts under the `servers` group, ensuring a home directory, sudoers drop-in, and (when present) the provided SSH key.

## Prerequisites
- Control node has access to `inventory/hosts.yml`'s private SSH key (`ansible_ssh_private_key_file`).
- Your account can become `root` on the target host (`become: true` is enabled).
- Each user entry includes a public key (`ssh-ed25519 …`) for installing access when `state: present`.

## Layout
- `inventory/hosts.yml` lists servers and connection details (e.g., `server1` uses `~/.ssh/id_ed25519_uapp`).
- `inventory/host_vars/<host>.yml` defines a `users` list; each item includes `name`, `key`, and `state` (`present` / `absent`).
- `playbooks/manage_users.yml` loops over `users`, adding the account when `state: present`, removing it when `state: absent`, and ensuring sudoers/keys only for present users.

## Managing a developer account
1. Open `inventory/host_vars/<host>.yml` for the target server.
2. Add or update the developer entry in the `users` list:
   ```yaml
   users:
     - name: developer
       key: "ssh-ed25519 AAAAB3NzaC1yc2EAAAADAQABAAABAQC..."
       state: present
   ```
3. To remove the account, set `state: absent` for that entry.
4. Optionally limit hosts with `--limit` or pass a different `users` list via `-e users=@users.yml`.

## Running the playbook
- Default execution:
  ```
  ansible-playbook -i inventory/hosts.yml playbooks/manage_users.yml
  ```
- Target one host:
  ```
  ansible-playbook -i inventory/hosts.yml playbooks/manage_users.yml --limit server1
  ```
- Provide a custom list of users:
  ```
  ansible-playbook -i inventory/hosts.yml playbooks/manage_users.yml -e users=@custom_users.yml
  ```

## Testing and verification
1. Dry run with `--check --limit` before applying:
   ```
   ansible-playbook -i inventory/hosts.yml playbooks/manage_users.yml --check --limit server1
   ```

## Example `inventory/host_vars/server1.yml`
```yaml
users:
  - name: user1
    key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHJRn9SnhdlfnLj/PRS5hW4HrZgf7OfsUweVvXv99Ld7 user1@example.com"
    state: present
```
