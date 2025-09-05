Role Name
=========

[![Ansible Lint](https://github.com/arnlec/cloud-native-setup/actions/workflows/ansible-lint.yml/badge.svg)](https://github.com/arnlec/cloud-native-setup/actions/workflows/ansible-lint.yml)

This role allow to deploy a cloud native environment based on K3S 

Requirements
------------

N/A

Role Variables
--------------

* argocd_enabled (default: false)
* argocd_domain
* argocd_cert
* argocd_key
* flux_enabled (default: false)
* flux_version: (defaut: 2.6.4)



Dependencies
------------

* xanmanning.k3s

Example Playbook
----------------

    - hosts: servers
      vars:
        k3s_version: v1.31.1+k3s1
        k3s_become: true
        argocd_domain: "argocd.k3s.dev"
        argocd_cert: "{{ lookup('file', './certs/argocd.k3s.dev.crt') }}"
        argocd_key: "{{ lookup('file', './certs/argocd.k3s.dev.key') }}"
      
      roles:
         - cloud-native-setup

License
-------

BSD

Author Information
------------------

## Development

### Code Quality

This repository uses [ansible-lint](https://ansible-lint.readthedocs.io/) to ensure code quality and Ansible best practices. The linting process runs automatically on every push via GitHub Actions.

To run ansible-lint locally:

```bash
# Install ansible-lint
pip install ansible-lint

# Run linting
ansible-lint .
```

The ansible-lint configuration can be found in `.ansible-lint` file.


