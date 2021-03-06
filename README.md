[![Build Status](https://travis-ci.org/chouseknecht/minishift-up-role.svg?branch=master)](https://travis-ci.org/chouseknecht/minishift-up-role)

# minishift-up-role

Installs [minishift](https://github.com/minishift/minishift), and the [OpenShift client](https://github.com/openshift/origin), and creates a running minishift instance.

Performs the following tasks:

- Downloads and installs the latest minishift
- Downloads and installs the OpenShift client
- Installs the Docker Machine driver
- Creates a minishift instance 
- Grants cluster admin to the *developer* account
- Adds a persistent volume

Supported platforms: 

- Debian
- Red Hat
- OSX

## Prerequisites 

- Prior to running the role, clear your terminal session of any DOCKER* environment variables.
- sudo access is required for installing packages 

**OSX**

- [homebrew](https://brew.sh) 
- [Ansible 2.2+](https://docs.ansible.com)


**Linux**

- KVM installed and working. The role installs the Docker Machine driver for KVM, but it assumes KVM is already installed, and working.
- Ansible 2.2+


**Fedora**

- Install packages python2-dnf, and libselinux-python

## Known Issues

**Fedora**

- Before accessing the Docker daemon on the minishift instance, you'll need to modify the `/etc/sysconfig/docker` script to prevent it from overriding the DOCKER_CERT_PATH environment variable. Edit the file, and change the line `DOCKER_CERT_PATH=/etc/docker` to the following:

    ```
    if [ -z "${DOCKER_CERT_PATH}" ]; then
        DOCKER_CERT_PATH=/etc/docker
    fi
    ```
**Linux/OSX**

- After running `eval $(minishift docker-env)` to set the environment, change the value of DOCKER_API_VERSION to match the version of Docker installed on the minishift instance by running the following:

    ```
    $ export DOCKER_API_VERSION=1.22
    ```

## Defaults

**minishift_repo:** minishift/minishift

> Repo where the minishift binary can be found

**minishift_github_url:** https://api.github.com/repos

> URL to access GitHub API. 

**minishift_release_tag_name:** "v1.0.0-beta.1"

> The minishift release to install.

**minishift_dest:** /usr/local/bin**

> Where to install the minishift binary.

**minishift_force_install:** yes

> Overwrite any existing minishift binary found at {{ minishift_dest }}

**minishift_volume:**

```
   name: pv0001
   path: /home/docker/pv0001/
   size: 5Gi
```

>  Object of volume attributes including the following:

**minishift_restart:** yes

> Stop and recreate the existing minishift instance.

**minishift_delete:** no

> Perform `minishift delete`, and delete `~/.minishift`.

**minishift_start_options:**

```
  - insecure-registry 172.30.0.0/16
  - insecure-registry minishift
  - iso-url https://github.com/minishift/minishift-centos-iso/releases/download/v1.0.0-alpha.1/minishift-centos.iso
```

> Options to pass to `minishift start`.

**openshift_repo:** openshift/origin

> GitHub repo for retrieving the OpenShift client.

**openshift_client_dest:** /usr/local/bin

> Where to install the OpenShift client binary.

**openshift_force_client_install:** yes

> Overwrite any existing OpenShift client binary found at {{ openshift_client_dest }}. 

## Example Playbook

Below is a sample playbook that includes all of the default parameters. You'll find this exact example in [files/minishift-up.yml](files/minishift-up.yml):

```
---
- name: Install minishift
  hosts: localhost
  connection: local
  gather_facts: yes
  roles:
    - role: chouseknecht.minishift-up-role
      minishift_repo: minishift/minishift 
      minishift_github_url: https://api.github.com/repos
      minishift_release_tag_name: "v1.0.0-beta.1"
      minishift_dest: /usr/local/bin  
      minishift_force_install: yes
      minishift_volume:
        name: pv0001
        path: /data/pv0001/
        size: 5Gi
      minishift_restart: yes 
      minishift_delete: no
      minishift_start_options:
      - insecure-registry 172.30.0.0/16
      - insecure-registry minishift
      - iso-url https://github.com/minishift/minishift-centos-iso/releases/download/v1.0.0-alpha.1/minishift-centos.iso
      openshift_repo: openshift/origin
      openshift_client_dest: /usr/local/bin
      openshift_force_client_install: yes
```

After you install the role, copy *file/minishift-up.yml* to your project directory, and execute it with the `--ask-sudo-pass` option. For example:

```
# Install the role 
$ ansible-galaxy install chouseknecht.minishift-up-role

# Copy the playbook from your roles path to the current working directory 
$ cp ${ANSIBLE_ROLES_PATH}/chouseknecht.minishift-up-role/files/minishift-up.yml .

# Create a localhost inventory file
$ echo "localhost">./inventory

# Run the playbook
$ ansible-container -i inventory --ask-sudo-pass minishift-up.yml
```

## Dependencies

None

## License

Apache v2

## Author 

[@chouseknecht](https://github.com/chouseknecht)
