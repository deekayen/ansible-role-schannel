# deekayen.schannel

[![CI](https://github.com/deekayen/ansible-role-schannel/actions/workflows/ci.yml/badge.svg)](https://github.com/deekayen/ansible-role-schannel/actions/workflows/ci.yml) [![Ansible Galaxy](https://img.shields.io/badge/galaxy-deekayen.schannel-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/deekayen/schannel/) [![Project Status: Inactive – The project has reached a stable, usable state but is no longer being actively developed; support/maintenance will be provided as time allows.](https://www.repostatus.org/badges/latest/inactive.svg)](https://www.repostatus.org/#inactive) ![BSD 3-Clause license](https://img.shields.io/badge/license-BSD%203--Clause-blue) ![Windows platform](https://img.shields.io/badge/platform-windows-lightgrey)

An Ansible role that sets Windows Schannel registry values to choose which protocols, ciphers, hashes, and Diffie-Hellman key sizes the OS offers to services such as IIS, RDP, and WinRM. It also sets `SchUseStrongCrypto` so .NET Framework 4.x applications use the OS TLS settings.

Every change is a `ansible.windows.win_regedit` write under `HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL` or `HKLM:\SOFTWARE\[Wow6432Node\]Microsoft\.NETFramework\v4.0.30319`. Tasks run only when `ansible_facts.os_family == 'Windows'`, so a mixed inventory skips non-Windows hosts.

## Requirements

- ansible-core 2.15 or newer on the controller.
- The `ansible.windows` collection: `ansible-galaxy collection install ansible.windows`.
- A WinRM or SSH connection to the target with administrative rights, since every key is under `HKLM`.
- Fact gathering left on. The role branches on `ansible_facts.os_family` and `ansible_facts.os_name`.

Schannel reads these keys at boot. **The role does not reboot the host**, so the new settings take effect after the next restart. Pair it with `ansible.windows.win_reboot` or a maintenance window.

## Supported platforms

`meta/main.yml` declares Windows, all versions. CI lints the role and runs `ansible-playbook --syntax-check`; it does not apply the role to a Windows host.

## Installation

From Ansible Galaxy:

```bash
ansible-galaxy role install deekayen.schannel
ansible-galaxy collection install ansible.windows
```

Or pin it in `requirements.yml`:

```yaml
---
roles:
  - name: deekayen.schannel
    src: https://github.com/deekayen/ansible-role-schannel.git
    scm: git
    version: main

collections:
  - name: ansible.windows
```

```bash
ansible-galaxy install -r requirements.yml
```

## Role variables

Every variable is a boolean unless noted. `true` enables the item; `false` disables it.

### Protocols

| Variable | Default | Registry effect |
| --- | --- | --- |
| `schannel_pct` | `false` | `Protocols\PCT 1.0\Server\Enabled` |
| `schannel_sslv2` | `false` | `Protocols\SSL 2.0\{Client,Server}` |
| `schannel_sslv3` | `false` | `Protocols\SSL 3.0\{Client,Server}` |
| `schannel_tlsv10` | `false` | `Protocols\TLS 1.0\{Client,Server}` |
| `schannel_tlsv11` | `true` | `Protocols\TLS 1.1\{Client,Server}` |
| `schannel_tlsv12` | `true` | `Protocols\TLS 1.2\{Client,Server}` |

Client keys get `DisabledByDefault`; server keys get `Enabled`, plus `DisabledByDefault` for TLS 1.1 and 1.2. The role does not manage TLS 1.3 keys. `tasks/assert.yml` fails the play if TLS 1.0, 1.1, and 1.2 are all `false`.

### Ciphers

| Variable | Default | Ciphers |
| --- | --- | --- |
| `schannel_aes_128` | `true` | AES 128/128 |
| `schannel_aes_256` | `true` | AES 256/256 |
| `schannel_3des` | `false` | Triple DES 168 (Triple DES 168/168 on Windows Server 2008 and Windows 7) |
| `schannel_des` | `false` | DES 56/56 |
| `schannel_null` | `false` | NULL |
| `schannel_rc2` | `false` | RC2 40/128, 56/128, 128/128 |
| `schannel_rc4` | `false` | RC4 40/128, 56/128, 64/128, 128/128 |

### Hashes

| Variable | Default |
| --- | --- |
| `schannel_md5` | `false` |
| `schannel_sha1` | `true` |
| `schannel_sha256` | `true` |
| `schannel_sha384` | `true` |
| `schannel_sha512` | `true` |

### Key exchange and .NET

| Variable | Default | Description |
| --- | --- | --- |
| `schannel_dh_enabled` | `true` | Sets `KeyExchangeAlgorithms\Diffie-Hellman\Enabled` to `0xFFFFFFFF` or `0`. |
| `schannel_dhmodulus` | `2048` | Integer. Written to Diffie-Hellman `ServerMinKeyBitLength` and PKCS `ClientMinKeyBitLength`. Must be `1024`, `2048`, `3072`, or `4096`; the role asserts this. |
| `schannel_usestrongcrypto` | `true` | Adds `SchUseStrongCrypto=1` under both the 64-bit and `Wow6432Node` .NET Framework v4.0.30319 keys when `true`, removes it when `false`. |

## Behavior

- Before setting cipher values, the role deletes keys named without the key-length suffix (`AES 128`, `RC4 40`, and so on). Older releases of this role created those by mistake.
- On releases newer than Windows Server 2008 and Windows 7, a `win_shell` task pre-creates the `Cipher/Bits` subkeys so `win_regedit` treats the slash as part of the key name. That task reports `changed` on every run.
- On Windows Server 2008 and Windows 7, the role removes `Triple DES 168` and writes `Triple DES 168/168`; newer releases use `Triple DES 168`.

## Dependencies

None. The `ansible.windows` collection is a requirement, not a role dependency.

## Example playbook

Keep TLS 1.0 on for a legacy client, then reboot to apply:

```yaml
---
- name: Harden Windows Schannel settings.
  hosts: iis_servers

  vars:
    schannel_tlsv10: true
    schannel_dhmodulus: 3072

  roles:
    - deekayen.schannel

  post_tasks:
    - name: Reboot so Schannel picks up the new registry values.
      ansible.windows.win_reboot:
```

## Tags

Every task has its own tag, so `--tags` or `--skip-tags` can apply a subset.

| Scope | Tags |
| --- | --- |
| Whole role | `schannel`, `security`, `windows`, `main` |
| Groups | `protocols`, `ciphers`, `hashes` |
| Per protocol | `pct10`, `ssl20_client`, `ssl20_server`, `ssl30_client`, `ssl30_server`, `tls10_client`, `tls10_server`, `tls11_client`, `tls11_server`, `tls11_state`, `tls12_client`, `tls12_server`, `tls12_state` |
| Per hash | `md5`, `sha1`, `sha256`, `sha384`, `sha512` |
| Per cipher | `aes128`, `aes256`, `aes_128_128`, `aes_256_256`, `des56`, `des_56_56`, `null`, `rc2_40`, `rc2_56`, `rc2_128`, `rc2_40_128`, `rc2_56_128`, `rc2_128_128`, `rc4_40`, `rc4_56`, `rc4_64`, `rc4_128`, `rc4_40_128`, `rc4_56_128`, `rc4_64_128`, `rc4_128_128`, `3des_168`, `set_3des_168_168`, `remove_3des_168`, `seed_cipher_keys` |
| Key exchange | `dh_exchange`, `dh_modulus`, `dh_kb3174644` |
| .NET | `dotnet`, `wow6432node` |

Input validation in `tasks/assert.yml` is tagged `always`.

## Development

CI runs on every push to `main` and every pull request (see `.github/workflows/ci.yml`). It installs `ansible.windows`, runs `ansible-lint --profile production`, and syntax-checks `tests/test.yml`. To run the same checks locally:

```bash
pip3 install ansible-lint
ansible-galaxy collection install ansible.windows
ansible-lint --profile production
mkdir -p .ansible/roles && ln -sfn "$PWD" .ansible/roles/deekayen.schannel
ANSIBLE_ROLES_PATH=.ansible/roles ansible-playbook --syntax-check tests/test.yml -i tests/inventory
```

The repository also has a `.pre-commit-config.yaml`; run `pre-commit run --all-files` before pushing.

### Repository layout

| Path | Purpose |
| --- | --- |
| `tasks/main.yml` | Runs input validation, then `windows.yml` on Windows hosts. |
| `tasks/assert.yml` | Checks `schannel_dhmodulus` and that one TLS version stays on. |
| `tasks/windows.yml` | Imports the three files below, then sets key exchange and .NET values. |
| `tasks/windows/protocols.yml` | Protocol client and server keys. |
| `tasks/windows/ciphers.yml` | Cipher keys. |
| `tasks/windows/hashes.yml` | Hash keys. |
| `defaults/main.yml` | Every user-facing variable. |
| `tests/` | Syntax-check playbook and inventory used by CI. |

## Releases

Pushing a git tag runs `.github/workflows/release.yml`, which imports the tagged commit into Ansible Galaxy as `deekayen.schannel`. The import needs a `GALAXY_API_KEY` repository or organization secret.

## References

- [Restrict cryptographic algorithms and protocols in Schannel.dll](https://learn.microsoft.com/en-us/troubleshoot/windows-server/windows-security/restrict-cryptographic-algorithms-protocols-schannel)
- [TLS/SSL settings in the Schannel SSP technical reference](https://learn.microsoft.com/en-us/windows-server/security/tls/tls-registry-settings)
- [Microsoft Security Advisory 3174644: Updated support for Diffie-Hellman key exchange](https://learn.microsoft.com/en-us/security-updates/securityadvisories/2016/3174644)
- [MS16-065: Security update for .NET Framework (CVE-2016-0149)](https://learn.microsoft.com/en-us/security-updates/securitybulletins/2016/ms16-065)

## License

BSD 3-Clause. See [LICENSE](LICENSE).

## Author

[David Norman](https://github.com/deekayen). Sponsorship links are in [.github/FUNDING.yml](.github/FUNDING.yml).
