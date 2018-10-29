## CentOS 7 PXE Server Installation and Configuration Role

This role installs and configures a PXE BOOT service. It assumes that an existing ISO image(s) is already present on the server in `/isos` directory.

It sets up a local httpd server to provide installation packages for remote BOOTP clients.

## Requirements

None.

## Role Variables

Add additional ISO in `defaults/main.yml` by adding:

```ini
- name: distro name
  iso_location: path to the ISO image on the server
  mount_point: where to mount it for the web server
```

## Dependencies

None.

## Example Playbook

Fetch this role from Ansible Galaxy:

`ansible-galaxy install mariuszczyz.centos-pxe`


In playbook.yml:

```
- hosts: servers
  roles:
    - { role: mariuszczyz.centos-pxe, tags: ['centos-pxe'] }
```
Run it:

`ansible-playbook -i hosts playbook.yml --user root --ask-pass --limit=servers`

## License

BSD

## Author Information

Author: Mariusz Czyz
Date: 10/2018
