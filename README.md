Role Name
=========

This role allow to deploy a cloud native environment based on K3S 

Requirements
------------

N/A

Role Variables
--------------

* argocd_domain
* argocd_cert
* argocd_key


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


