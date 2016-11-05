# development-vm

## Goals
* Provide a base-line VM definition for development work.  Especially useful for 
  Windows-based developers since tools like Ansible are unavailable on that platform.

## Prerequisites 
* [Git](https://git-scm.com/)
* [Virtualbox](https://www.virtualbox.org/)
* [Vagrant](https://www.vagrantup.com)

## Setup
* First off decide where you will store your development VM checkout and configuration. 
  I usually create a dedicated VM directory for all my virtual machines.
  ```powershell
  E:\vm> 
  ```

* Clone the project.  I typically clone into a directory named after the hostname of the VM I am creating.
  ```powershell
  E:\vm>
  git clone git@github.com:bigj64/development-vm.git development-vm.localhost
  git submodule update --init
  ```

* Before you can actually bring up the VM you will need to define a couple of configuration
  files.  Their specific formats and values are defined below but make sure you have these 
  defined before continuing.
  ```powershell
  E:\vm\development-vm.localhost>
  cp vagrant_config.yml.dist vagrant_config.yml
  cp ansible/master.yml.dist ansible/master.yml
  cp ansible/extra_vars.yml.dist ansible/extra_vars.yml
  ```

* Once your configs are defined all you have to do is bring up the VM
  ```powershell
  E:\vm\development-vm.localhost>
  vagrant up
  ```

* Assuming all the provisioning went as planned you should now be good to go!  If 
  you encounter configuration related errors during provisioning you can fix them and 
  try again as follows:
  ```powershell
  E:\vm\development-vm.localhost>
  vagrant provision
  ```

## Troubleshooting

## Config

### vagrant_config.yml

Defines variables used in Vagrantfile.

```yaml
---
config:
  hostname: development-vm.local
  vmname: development-vm
  sshport: 22222
  publicmac: DEADBEEF1234
  memory: 2048
  gui: (true|false)
  ansible_version: 2.2.0.0
```

* **hostname**: _(Required)_ Value of the config.vm.hostname option.  Any valid hostname.
* **vmname**: _(Required)_ Value of the config.vm.provider.name option.  Any valid VM name.
* **sshport**: _(Optional)_ Overrides the default SSH port forwarding.  Useful if you want
  bind this VM to a specific port.  Must be a valid port number or omitted.
* **publicmac**: _(Optional)_ Allows explicitly setting MAC address on public adapter.
  This was added to support networking rules on persistent machines.  Must be a valid MAC
  address, auto, or omitted.
* **memory**: _(Optional)_ How much memory to give this VM (in MB).
* **gui**: _(Optional)_ True to start with a visible VM GUI.  False to run in headless mode.
* **ansible_version**: _(Optional)_ Set the version of anisble to be installed via pip.

### master.yml

Defines which Ansible plays will be run on the development VM.

```yaml
---
- hosts: localhost
# - include: playbook/someplay.yml
...
```

Uncomment the playbooks you would like to apply.

### extra_vars.yml

Defines variables used during Ansible provisioning.

```yaml
---
development_vm:
  # See Roles for valid config options
```

## Roles

### apt-packages
Installs aptitude packages from upstream repositories
#### Config
```yaml
apt_packages:
  - foo
  - bar
```
apt_packages is a list taking any number of aptitude package names

### composer
Installs composer binary
#### Config
None

### composer-global-packages
Installs composer packages globally
#### Config
```yaml
composer_global_packages:
  vendor/foo:
    bin: foo
    version: dev-master
  vendor/bar:
    bin: bar
    version: 1.*
```
composer_global_packages is a hash, keys are Composer vendor/package names.  Each vendor/package element is a hash defined as follows:

* **bin**: (_Required_) Name of the binary being created.
* **version**: (_Required_) Composer package version.

### deb-packages
Downloads and installs debian packages from arbitrary URLs
#### Config
```yaml
deb_packages:
  foo:
    url: http://location.of/foo.deb
    dest: /tmp/foo.deb
  bar:
    url: http://location.of/bar.deb
    dest: ~/bar.dev
```
deb_packages is a hash, keys are arbitrary identifiers.  Each deb identifier element is a hash defined as follows:

* **url**: (_Required_) URL of the Debian package file.
* **dest**: (_Required_) Download destination.

### groups
Adds additional groups
#### Config
```yaml
groups:
  groupname: 123
```
groups is a hash, keys are group names, values are optional GIDs.

### java-dev
Configures system for Java development
#### Config
```yaml
java_dev:
  java: openjdk
  java_home: /path/to/openjdk
```
java_dev is a hash defined as follows:

* **java**: (_Required_) Aptitude package name of the Java JDK to be installed
* **java_home**: (_Required_) Path to the JDK for setting JAVA_HOME env variable

### java-dev-maven
Configures system for Java development using Maven
#### Config
```yaml
java_dev_maven:
  maven: maven
  settings: http://location.of/settings.xml
```
java_dev_maven is a hash defined as follows:

* **maven**: (_Required_) Aptitude package name of Maven 
* **settings**: (_Optional_) URL download location of Maven settings

### java-dev-nexus
Configures system for Java development using Nexus.  Adds certificate too cacerts trust store, replacing existing certificate aliases when found.
#### Config
```yaml
java_dev_nexus:
  certificate: http://location.of/nexus.cer
  alias: nexus
  storepass: changeit
```
java_dev_nexus is a hash defined as follows:

* **certificate**: (_Required_) URL download location of Nexus server certificate
* **alias**: (_Required_) Certificate alias to be added to cacerts trust store
* **storepass**: (_Optional_) Keystore password for cacerts trust store

### network
Configures network adapters according to role template
#### Config
```yaml
network:
  gateway: ip.of.default.gateway
```
network is a hash defined as follows:

* **gateway**: (_Optional_) IP of your prefered default gateway

### nfs-client
Configures and mounts NFS shares using autofs.  Creates mount points if needed.
#### Config
```yaml
nfs_client:
  /path/to/mount:
    server: location.of.nfs.server
    export: /path/to/export
    options: rw
```

nfs_client is a hash, keys are paths to mount points on the guest.  Each mount point element is a hash defined as follows:

* **server**: _(Required)_ URL or IP address to the NFS server
* **export**: _(Required)_ Path to the NFS export on the remote server
* **options**: _(Optional)_ NFS options to be passed to mount / stored in fstab

### timezone
Sets the local timezone
#### Config
```yaml
timezone: 
  name: America/New_York
```

timezone is a hash, keys are defined as follows:

* **name**: (_Required_) This may be any valid tz database identifier as defined by IANA: [IANA Timezones](http://www.iana.org/time-zones).

### user
Configures user and group.

**Note**: All users created start with a default password of "password". Each user will be required to change it on first login.
#### Config
```yaml
user:
  username: foo
  usergroup: foo
  groups: bar,baz
  uid: 10000
  gid: 10000
  sshkey: ssh-rsa <really long key here>
```

user is a hash, keys are defined as follows:

* **username**: _(Required)_ Any valid username
* **usergroup**: _(Optional)_ Defaults to same as username
* **groups**: _(Optional)_ Additional groups for this user
* **uid**: _(Optional)_ Any valid user id
* **gid**: _(Optional)_ Any valid group id
* **sshkey**: _(Optional)_ Valid ssh key public string
