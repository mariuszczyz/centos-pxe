# PXE Boot Server Ansible Role

This role builds a PXE boot server. It configures TFTP with the syslinux
bootloader, serves installation trees through Apache, and deploys Kickstart
files for unattended installs. Supported targets: Enterprise Linux 9,
Enterprise Linux 10, and Fedora.

## Platform matrix

The role is tested on these operating systems:

| OS | Versions |
| --- | --- |
| Rocky Linux | 9, 10 |
| AlmaLinux | 9, 10 |
| CentOS Stream | 9, 10 |
| Fedora | 43, 44 |

## Requirements

- Apache HTTP Server preinstalled. The role writes files into
  `/etc/httpd/conf.d/` and restarts `httpd`. The
  [mariuszczyz.centos_apache](https://galaxy.ansible.com/mariuszczyz/centos_apache)
  Galaxy role installs it.
- A working DHCP server. Point `next-server` and the boot file name at the
  PXE host. The
  [mariuszczyz.centos_dhcpd](https://galaxy.ansible.com/mariuszczyz/centos_dhcpd)
  Galaxy role configures one.
- At least 2 GB of RAM on PXE-installed guests. The EL9 and EL10 installer
  fails with less. You can lower the allocation after the install.

## Role Variables

`meta/argument_specs.yml` validates input and documents every variable.
`defaults/main.yml` carries the applied default values. User variables
override them.

| Variable | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `pxe_isos_path` | str | no | `/isos` | Directory where installation ISO images are stored. |
| `pxe_isos` | list of dict | yes | three EL9/EL10/Fedora entries | Installation ISO images to download, mount, and serve. |
| `pxe_kickstarts` | list of dict | yes | three base entries | Kickstart files to deploy and to list in the PXE menu. |
| `pxe_kickstart_hashed_root_password` | str | yes | none | Hashed root password for Kickstart files. Generate with `openssl passwd -6`. |
| `pxe_non_root_user_name` | str | yes | none | Non-root administrator user created by Kickstart installs. |
| `pxe_non_root_user_password` | str | yes | none | Hashed password for the non-root user. Generate with `openssl passwd -6`. |
| `pxe_timezone` | str | yes | none | System timezone for Kickstart installs, for example `America/Chicago`. |
| `pxe_ntp_servers` | str | no | `""` | Comma-separated NTP servers for Kickstart installs. Empty leaves installer defaults. |
| `pxe_el9_hostname` | str | no | `el9.home.local` | Hostname set by the EL9 Kickstart file. |
| `pxe_el9_install_url` | str | no | `http://<host>/el9` | Repository URL the EL9 installer reads packages from. |
| `pxe_el10_hostname` | str | no | `el10.home.local` | Hostname set by the EL10 Kickstart file. |
| `pxe_el10_install_url` | str | no | `http://<host>/el10` | Repository URL the EL10 installer reads packages from. |
| `pxe_fedora_hostname` | str | no | `fedora.home.local` | Hostname set by the Fedora Kickstart file. |
| `pxe_fedora_install_url` | str | no | `http://<host>/fedora` | Repository URL the Fedora installer reads packages from. |

`pxe_isos` entries use these keys. All are required strings:

| Key | Description |
| --- | --- |
| `name` | Short OS name. Used in TFTP and HTTP paths. |
| `download_url` | URL to download the ISO image from. |
| `iso_path` | Local path of the ISO image. |
| `mount_point` | Mount point. Must live under `/var/www/html` so Apache serves it. |

`pxe_kickstarts` entries use these keys. All are required strings:

| Key | Description |
| --- | --- |
| `name` | Kickstart template base name. Must match `templates/<name>.cfg.j2`. |
| `iso` | `pxe_isos` entry that provides the kernel and initrd for this entry. |
| `label` | PXE boot menu label. |

The default `pxe_isos` list downloads Rocky 9, Rocky 10, and Fedora 44 ISO
images. The installed `<host>` value comes from the primary IPv4 address of
the PXE server.

## Kickstart notes

Generate password hashes with `openssl passwd -6`. The command prompts for
the password and prints the hash:

```shell
$ openssl passwd -6
Password:
Verifying - Password:
$6$gdGbs42fZoKUVwQH$eY2nId.oONxK9MneuM58Vg2NPEuftngWmwfK09YW4DQLs3Hcq5F5HEohDEcM.Ci3p8gQrVuygTfScim7MY6QI1
```

The shipped templates assume all of the following:

- automatic partitioning with LVM
- SELinux disabled
- firewall disabled
- DHCP network configuration
- one administrator user in the `wheel` group
- text mode install with a reboot at the end

Review and adjust the templates before production use.

## Breaking changes in this release

- All role variables renamed to `pxe_` snake_case. For example, `ISOS` is
  now `pxe_isos` and `TIMEZONE` is now `pxe_timezone`.
- The role now enables `tftp.socket` through systemd socket activation. The
  xinetd service is gone. xinetd does not exist on Fedora 34 and later.
- CentOS 7 and CentOS 8 support dropped. The templates for them are deleted.
- Kickstart parameters are required: `pxe_kickstart_hashed_root_password`,
  `pxe_non_root_user_name`, `pxe_non_root_user_password`, `pxe_timezone`.
  The role fails fast when one is missing.
- Fixed the Apache autoindex config destination. The old role copied it to
  `/etc/httpd/conf.d/autoindex.conf.conf`.

## Testing

`molecule test` runs the full sequence on all eight platforms in the matrix
with Podman. ISO downloads are disabled in tests. The kickstart files and
the PXE menu still render.

The EL10 images need a CPU with x86-64-v3 support (Haswell generation or
newer). On older CPUs the EL10 containers fail to start.

Prerequisites:

```shell
pip install --user molecule "molecule-plugins[podman]"
ansible-galaxy collection install -r requirements.yml
```

Run the tests from the repository root:

```shell
molecule test
```

## Example Playbook

```yaml
- hosts: pxe_servers
  become: true

  roles:
    - role: centos_pxe
```

## Dependencies

- [mariuszczyz.centos_apache](https://galaxy.ansible.com/mariuszczyz/centos_apache):
  installs Apache.
- [mariuszczyz.centos_dhcpd](https://galaxy.ansible.com/mariuszczyz/centos_dhcpd):
  optional DHCP server for the PXE network.

## License

BSD-3-Clause

## Author Information

Mariusz Czyz
