# ansible-role-lldap

Installs and configures [lldap](https://github.com/lldap/lldap) — a lightweight
authentication server that provides an opinionated, simplified LDAP interface
for authentication — as a systemd service on Debian/Ubuntu/RHEL-family hosts.

The role downloads the prebuilt Linux binary from the lldap GitHub releases
page, installs it under `/opt/lldap`, renders `lldap_config.toml`, and runs it
under a dedicated `lldap` system user via systemd.

## Requirements

- systemd-based target host
- Outbound internet access to GitHub releases (or point `lldap_download_url`
  at an internal mirror)

## Role Variables

See `defaults/main.yml` for the full list. The important ones:

| Variable | Default | Description |
|---|---|---|
| `lldap_version` | `0.6.1` | Release version to install |
| `lldap_install_dir` | `/opt/lldap` | Where the binary is installed |
| `lldap_config_dir` | `/etc/lldap` | Where `lldap_config.toml` lives |
| `lldap_data_dir` | `/var/lib/lldap` | SQLite DB / working directory |
| `lldap_ldap_base_dn` | `dc=example,dc=com` | Base DN |
| `lldap_ldap_user_dn` | `admin` | Initial admin username |
| `lldap_ldap_user_email` | `admin@example.com` | Initial admin email |
| `lldap_ldap_user_pass` | `""` | Initial admin password — **required**, set via vault |
| `lldap_jwt_secret` | `""` | JWT signing secret — **required**, set via vault |
| `lldap_key_seed` | `""` | Key seed for password hashing — **required**, set via vault |
| `lldap_http_port` | `17170` | Web UI / GraphQL API port |
| `lldap_ldap_port` | `3890` | LDAP listener port |
| `lldap_smtp_options_enabled` | `false` | Enable SMTP (password reset emails) |

`lldap_ldap_user_pass`, `lldap_jwt_secret`, and `lldap_key_seed` have no
default and the role will fail with an assertion error if they are left
blank — generate strong random values and store them in vault-encrypted
host/group vars.

## Example Playbook

```yaml
- hosts: lldap_servers
  become: true
  roles:
    - role: lldap
      vars:
        lldap_ldap_base_dn: "dc=mycorp,dc=io"
        lldap_ldap_user_pass: "{{ vault_lldap_admin_pass }}"
        lldap_jwt_secret: "{{ vault_lldap_jwt_secret }}"
        lldap_key_seed: "{{ vault_lldap_key_seed }}"
```

Generate secrets, e.g.:

```sh
openssl rand -base64 32   # jwt_secret / key_seed
```

## License

MIT
