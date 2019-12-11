# CentOS 7/8 & Fedora PXE Boot Unattended OS Installation and Configuration Role

This role configures the basic framework for a local PXE boot
environment allowing to perform unattended installation of CentOS 7,
CentOS 8, Fedora 31 and more (with custom configuration).

It installs and configures the following:

`Installation ISOs & Local RPM Repository` - if the installation ISOs do not
exist locally they will be downloaded and mounted. Once mounted, their content
will be used to generate a local RPM mirror used during the installation process.
The mirror can be also used later for additional RPM packages installations.

`TFTP server & PXE` - we need this to allow the network clients to boot via PXE.

This role pre-configures the PXE service with the following:

- Boot from local drive. Do not install anything
- Install Fedora 31 Manually with graphical GUI
- Install CentOS 7 Manually with graphical GUI
- Install CentOS 8 Manually with graphical GUI
- Base Unattended Fedora 31 Kickstart Install in Text Mode
- Base Unattended CentOS 7 Kickstart Install in Text Mode
- Base Unattended CentOS 8 Kickstart Install in Text Mode

It also installs all the necessary kernel images needed for remote PXE clients
to boot up properly.

`Apache Web Server` - with very little pre-configuration it'll be used
to create a locally accessible server-generated directory listings of
all RPM packages.

The local mirror will immitate the same directory structure as publically
available mirrors.

They will be accessible locally at:
(replace hostname.localdomain with own address)

- <http://hostname.localdomain/fedora31>
- <http://hostname.localdomain/centos7>
- <http://hostname.localdomain/centos8>

`Kickstart Files` - this role deploys CentOS7/8 & Fedora 31 unattended installation
Anaconda Kickstart files from templates. They are placed in a kickstart directory
in the web server root directory and are accessible by clients.

- <http://hostname.localdomain/kickstart/>

## Additional Notes

When building a new VM in VirtualBox or KVM allocate a minimum of 2 GB of RAM
for the guest. The CentOS installation process will most likely fail if less
than that is used. The amount of RAM can be lowered following a successfull
installation.

The end user is encouraged to review and customize the Kickstart configuration
templates. In their current form they are very basic. All of them assume the
following:

- automatic partitioning with LVM
- SELinux off
- firewall off
- minimal software selection
- root login allowed
- single non-root administrator user
- DHCP client network configuration

The idea for doing as little as possible with Anaconda and Kickstart
installation is to rely on post installation configuration customization.

## Requirements

### Apache

Standard installation of Apache web server is required in order for the
Kickstart process to access the installation packages locally.

A simple Apache role can be installed from Galaxy:

`ansible-galaxy install mariuszczyz.centos_apache`

- <https://galaxy.ansible.com/mariuszczyz/centos_apache>

### DHCPd

Working local DHCP service.

Alternatively, a dedicated DHCP can be set up on the PXE boot server by using
this role: [CentOS & Fedora DHCP Server Installation and Configuration Ansible
Role](https://galaxy.ansible.com/mariuszczyz/centos_dhcpd).

### Operating System Installation ISO Images

This role assumes the location of operating system installation ISO images is in
`/isos/`. Leave it as is or change it in the `defaults/main.yml`. However,
the ISO images must be downloaded prior to running this role. Otherwise,
it will not have access to all the files it needs to properly set up the
pre-boot environment. The task of downloading the ISOs has been purposely
left out of this role.

### Kickstart Files

The minimum changes needed for the Kickstart installation files to work:

`rootpw --iscrypted PASSWORD_HASH` - root password hash  

#### Instructions on how to create a Kickstart root password hash

Run this command on the CLI: `openssl passwd -6`

Available algorithm options:

```shell
 -6                  SHA512-based password algorithm
 -5                  SHA256-based password algorithm
 -apr1               MD5-based password algorithm, Apache variant
 -1                  MD5-based password algorithm
 -aixmd5             AIX MD5-based password algorithm
 -crypt              Standard Unix password algorithm (default)
```

It'll prompt for the password and output the hash:

_Note: not a real password below_

```shell
Password:
Verifying - Password:
$6$gdGbs42fZoKUVwQH$eY2nId.oONxK9MneuM58Vg2NPEuftngWmwfK09YW4DQLs3Hcq5F5HEohDEcM.Ci3p8gQrVuygTfScim7MY6QI1
```

The rest of the setting can be customized optionally to fit your own needs,
like partitioning, timezone, additional packages, etc.

## Role Variables

| Variable | Comment | Example |
| --- | ------- | ------- |
| ISOS_PATH | Directory where ISO installation images will be stored locally | /isos |
| NAME | Sperating system name | fedora31 |
| ISO_LOCATION | Full path to ISO image | /isos/CentOS-7-x86_64-Everything-1908.iso |
| MOUNT_POINT | Full path to where the ISO image should be mounted on the local file system | /var/www/centos7 |
| KICKSTART_HASHED_ROOT_PASSWORD | Kickstart hased root password. Use "pwkickstart" or "openssl passwd -6" to generate | bEzYf1S49$yu |
| NON_ROOT_USER_NAME | Non root admin user account to create on the new system | mariusz |
| NON_ROOT_USER_PASSWORD | Kickstart hased user password. Use "pwkickstart" or "openssl passwd -6" to generate | bEzYf1S49$yu |
| TIMEZONE | Local timezone | America/Chicago |
| NTP_SERVERS | Network time servers. Local or public. | ntp.localdomain |
| FEDORA_HOSTNAME | Default hostname for the new Fedora server | fedora31.localdomain |
| FEDORA_NETWORK_INSTALLATION_URL | This is where Anaconda will fetch Fedora packages from | <http://mirror.steadfastnet.com/fedora/releases/31/Everything/x86_64/os/> |
| CENTOS7_HOSTNAME | Default hostname for the new Fedora server | centos7.localdomain |
| CENTOS7_NETWORK_INSTALLATION_URL | This is where Anaconda will fetch CentOS packages from | <http://192.168.1.109/centos7> |
| CENTOS8_HOSTNAME | Default hostname for the new Fedora server | centos8.localdomain |
| CENTOS8_BASE_OS_URL | CentOS 8 BaseOS packages repository URL | <http://mirror.steadfastnet.com/centos/8/BaseOS/x86_64/os/> |
| CENTOS8_APPSTREAM_REPO_URL | CentOS 8 AppStream packages repository URL | <http://mirror.steadfastnet.com/centos/8/AppStream/x86_64/kickstart/> |

## Dependencies

[mariuszczyz.centos_apache](https://galaxy.ansible.com/mariuszczyz/centos_apache)
[mariuszczyz.centos_dhcpd](https://galaxy.ansible.com/mariuszczyz/centos_dhcpd)

Install the dependencies from Ansible Galaxy with `requirements.yml`

```yaml
# Install from Ansible Galaxy
- src: mariuszczyz.centos_apache
- src: mariuszczyz.centos_dhcpd
```

## Example Playbook

### Manual

Fetch this role from Ansible Galaxy manually:

`ansible-galaxy install mariuszczyz.centos_pxe`

### Not Manual

#### Galaxy

Or include this role from Ansible Galaxy via `requirements.yml`

```yaml
# requirements.yml
# Install from Ansible Galaxy
- src: mariuszczyz.centos_pxe
```

#### Github option

```yaml
# requirements.yml
# Install from Github repository
- src: https://www.github.com/mariuszczyz/centos_pxe
```

Then run this to install all dependencies from Ansible Galaxy:

`ansible-galaxy install -r requirements.yml`

### Run it

If you want to run this role individually create a new file:
`playbook.yml` (name it however you wish btw) with the following content:

```yaml
- hosts: servers
  user: YOUR USER
  become: True

  roles:
    - { role: mariuszczyz.centos_pxe, tags: ['centos_pxe'] }
```

Run it:

`ansible-playbook -i hosts playbook.yml`

## License

BSD

## Author Information

Author: Mariusz Czyz
Date: 12/2019
mariuszczyz.com
