# Secure Boot
Secure boot management.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_secure_boot/blob/main/meta/main.yml)

[collections/roles](https://github.com/r-pufky/ansible_secure_boot/blob/main/meta/requirements.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_secure_boot/blob/main/defaults/main)

## Dependencies
Part of the [r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv)
collection.

## Example Playbook
This will install MOK (Machine Owner Keys) and enable secure booting for
machines. A [high-level understanding of secure boot is required](https://github.com/r-pufky/ansible_secure_boot/blob/main/docs/README.md).

No additional management of DKMS. If custom MOK keys are used, it is the user's
responsibility to update the DKMS configuration to locate those keys.

Apply the base role
``` yaml
- name: 'Enable Secure Boot'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.secure_boot'
  vars:
    secure_boot_password: '{{ vault_password }}'
```
Machine is rebooted **only** if there are changes to secure boot;
`secure_boot_password` is needed at the physical console to confirm.

## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_srv/blob/main/docs/dev/environment/README.md)

Run all unit tests:
``` bash
molecule test --all
```

### Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_secure_boot/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
