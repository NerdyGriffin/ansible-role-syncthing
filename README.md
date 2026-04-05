# Ansible Role: nerdygriffin.syncthing

Installs and configures [Syncthing](https://syncthing.net/) from the official upstream repository, and handles automated two-pass device pairing across all hosts in a play.

## Requirements

- Ansible 2.14 or later
- Supported platforms: Ubuntu (jammy, noble), RHEL/Rocky/AlmaLinux 8 and 9
- `systemd` must be available on target hosts (Syncthing runs as a user systemd service)

## Role Variables

| Variable | Default | Description |
|---|---|---|
| `syncthing_user` | `setup` | OS user account that runs the Syncthing service |
| `syncthing_install_repo` | `true` | Add the official Syncthing apt/dnf repository before installing |
| `syncthing_folders` | `[]` | List of shared folders to configure (dicts with `id` and `path`) |
| `syncthing_api_key` | `""` | REST API key (set via vault in production; empty = skip) |
| `syncthing_gui_enabled` | `false` | Enable or disable the Syncthing web GUI |
| `syncthing_api_address` | `"127.0.0.1:8384"` | Listen address for the GUI/API |
| `syncthing_listen_address` | `"default"` | Listen address for device-to-device sync traffic |

### Folder list format

```yaml
syncthing_folders:
  - id: ansible
    path: /ansible
  - id: documents
    path: /home/setup/Documents
```

## Example Playbook

```yaml
---
- name: Deploy Syncthing on all Ansible controllers
  hosts: ansible_controllers
  become: true

  vars:
    syncthing_user: setup
    syncthing_api_key: "{{ vault_syncthing_api_key }}"
    syncthing_folders:
      - id: ansible
        path: /ansible

  roles:
    - role: nerdygriffin.syncthing
```

When the play targets multiple hosts, `tasks/pair_devices.yml` automatically pairs every host in the play with every other host and shares the configured folders between them. Both hosts must be present in the same play run for pairing to succeed (since each host's device ID is collected as a fact during the run).

## License

MIT

## Author

[NerdyGriffin](https://github.com/NerdyGriffin)
